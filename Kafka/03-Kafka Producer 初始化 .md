# Kafka Producer 初始化

## 初始化源码

```java
private KafkaProducer(ProducerConfig config, Serializer<K> keySerializer, Serializer<V> valueSerializer) {
	try {
		log.trace("Starting the Kafka producer");
		Map<String, Object> userProvidedConfigs = config.originals();
		this.producerConfig = config;
		this.time = new SystemTime();

		clientId = config.getString(ProducerConfig.CLIENT_ID_CONFIG);
		if (clientId.length() <= 0)
			clientId = "producer-" + PRODUCER_CLIENT_ID_SEQUENCE.getAndIncrement();
		Map<String, String> metricTags = new LinkedHashMap<String, String>();
		metricTags.put("client-id", clientId);
		MetricConfig metricConfig = new MetricConfig().samples(config.getInt(ProducerConfig.METRICS_NUM_SAMPLES_CONFIG))
				.timeWindow(config.getLong(ProducerConfig.METRICS_SAMPLE_WINDOW_MS_CONFIG), TimeUnit.MILLISECONDS)
				.tags(metricTags);
		List<MetricsReporter> reporters = config.getConfiguredInstances(ProducerConfig.METRIC_REPORTER_CLASSES_CONFIG,
				MetricsReporter.class);
		reporters.add(new JmxReporter(JMX_PREFIX));
		this.metrics = new Metrics(metricConfig, reporters, time);
		this.partitioner = config.getConfiguredInstance(ProducerConfig.PARTITIONER_CLASS_CONFIG, Partitioner.class);
		long retryBackoffMs = config.getLong(ProducerConfig.RETRY_BACKOFF_MS_CONFIG);
		this.metadata = new Metadata(retryBackoffMs, config.getLong(ProducerConfig.METADATA_MAX_AGE_CONFIG));
		this.maxRequestSize = config.getInt(ProducerConfig.MAX_REQUEST_SIZE_CONFIG);
		this.totalMemorySize = config.getLong(ProducerConfig.BUFFER_MEMORY_CONFIG);
		this.compressionType = CompressionType.forName(config.getString(ProducerConfig.COMPRESSION_TYPE_CONFIG));
		/* check for user defined settings.
		 * If the BLOCK_ON_BUFFER_FULL is set to true,we do not honor METADATA_FETCH_TIMEOUT_CONFIG.
		 * This should be removed with release 0.9 when the deprecated configs are removed.
		 */
		if (userProvidedConfigs.containsKey(ProducerConfig.BLOCK_ON_BUFFER_FULL_CONFIG)) {
			log.warn(ProducerConfig.BLOCK_ON_BUFFER_FULL_CONFIG + " config is deprecated and will be removed soon. " +
					"Please use " + ProducerConfig.MAX_BLOCK_MS_CONFIG);
			boolean blockOnBufferFull = config.getBoolean(ProducerConfig.BLOCK_ON_BUFFER_FULL_CONFIG);
			if (blockOnBufferFull) {
				this.maxBlockTimeMs = Long.MAX_VALUE;
			} else if (userProvidedConfigs.containsKey(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG)) {
				log.warn(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG + " config is deprecated and will be removed soon. " +
						"Please use " + ProducerConfig.MAX_BLOCK_MS_CONFIG);
				this.maxBlockTimeMs = config.getLong(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG);
			} else {
				this.maxBlockTimeMs = config.getLong(ProducerConfig.MAX_BLOCK_MS_CONFIG);
			}
		} else if (userProvidedConfigs.containsKey(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG)) {
			log.warn(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG + " config is deprecated and will be removed soon. " +
					"Please use " + ProducerConfig.MAX_BLOCK_MS_CONFIG);
			this.maxBlockTimeMs = config.getLong(ProducerConfig.METADATA_FETCH_TIMEOUT_CONFIG);
		} else {
			this.maxBlockTimeMs = config.getLong(ProducerConfig.MAX_BLOCK_MS_CONFIG);
		}

		/* check for user defined settings.
		 * If the TIME_OUT config is set use that for request timeout.
		 * This should be removed with release 0.9
		 */
		if (userProvidedConfigs.containsKey(ProducerConfig.TIMEOUT_CONFIG)) {
			log.warn(ProducerConfig.TIMEOUT_CONFIG + " config is deprecated and will be removed soon. Please use " +
					ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG);
			this.requestTimeoutMs = config.getInt(ProducerConfig.TIMEOUT_CONFIG);
		} else {
			this.requestTimeoutMs = config.getInt(ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG);
		}

		this.accumulator = new RecordAccumulator(config.getInt(ProducerConfig.BATCH_SIZE_CONFIG),
				this.totalMemorySize,
				this.compressionType,
				config.getLong(ProducerConfig.LINGER_MS_CONFIG),
				retryBackoffMs,
				metrics,
				time);
		List<InetSocketAddress> addresses = ClientUtils.parseAndValidateAddresses(config.getList(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG));
		this.metadata.update(Cluster.bootstrap(addresses), time.milliseconds());
		ChannelBuilder channelBuilder = ClientUtils.createChannelBuilder(config.values());
		NetworkClient client = new NetworkClient(
				new Selector(config.getLong(ProducerConfig.CONNECTIONS_MAX_IDLE_MS_CONFIG), this.metrics, time, "producer", channelBuilder),
				this.metadata,
				clientId,
				config.getInt(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION),
				config.getLong(ProducerConfig.RECONNECT_BACKOFF_MS_CONFIG),
				config.getInt(ProducerConfig.SEND_BUFFER_CONFIG),
				config.getInt(ProducerConfig.RECEIVE_BUFFER_CONFIG),
				this.requestTimeoutMs, time);
		this.sender = new Sender(client,
				this.metadata,
				this.accumulator,
				config.getInt(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION) == 1,
				config.getInt(ProducerConfig.MAX_REQUEST_SIZE_CONFIG),
				(short) parseAcks(config.getString(ProducerConfig.ACKS_CONFIG)),
				config.getInt(ProducerConfig.RETRIES_CONFIG),
				this.metrics,
				new SystemTime(),
				clientId,
				this.requestTimeoutMs);
		String ioThreadName = "kafka-producer-network-thread" + (clientId.length() > 0 ? " | " + clientId : "");
		this.ioThread = new KafkaThread(ioThreadName, this.sender, true);
		this.ioThread.start();

		this.errors = this.metrics.sensor("errors");

		if (keySerializer == null) {
			this.keySerializer = config.getConfiguredInstance(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
					Serializer.class);
			this.keySerializer.configure(config.originals(), true);
		} else {
			config.ignore(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG);
			this.keySerializer = keySerializer;
		}
		if (valueSerializer == null) {
			this.valueSerializer = config.getConfiguredInstance(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
					Serializer.class);
			this.valueSerializer.configure(config.originals(), false);
		} else {
			config.ignore(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG);
			this.valueSerializer = valueSerializer;
		}

		// load interceptors and make sure they get clientId
		userProvidedConfigs.put(ProducerConfig.CLIENT_ID_CONFIG, clientId);
		List<ProducerInterceptor<K, V>> interceptorList = (List) (new ProducerConfig(userProvidedConfigs)).getConfiguredInstances(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG,
				ProducerInterceptor.class);
		this.interceptors = interceptorList.isEmpty() ? null : new ProducerInterceptors<>(interceptorList);

		config.logUnused();
		AppInfoParser.registerAppInfo(JMX_PREFIX, clientId);
		log.debug("Kafka producer started");
	} catch (Throwable t) {
		// call close methods if internal objects are already constructed
		// this is to prevent resource leak. see KAFKA-2121
		close(0, TimeUnit.MILLISECONDS, true);
		// now propagate the exception
		throw new KafkaException("Failed to construct kafka producer", t);
	}
}
```



## 分析

- producerConfig - 配置信息

- clientId - `client.id`配置 ,如果没有配置则使用默认策略：producer-{自增长}（`AtomicInteger PRODUCER_CLIENT_ID_SEQUENCE = new AtomicInteger(1)`）

- partitioner - Kafka 分区信息

- retryBackoffMs -  `retry.backoff.ms` 配置的重试间隔时间，*默认100ms* ，避免在失败时进行过于频繁的重试

- metadata - 存储元数据的组件，`metadata.max.age.ms` 设置元数据的最大有效时间，*默认5min* ，过了最大有效时间会强制刷新元数据

- maxRequestSize - 最大请求大小，`max.request.size` 设置最大请求大小，以字节为单位，*默认1M*，也是单条记录的大小，需要注意的是这仅仅是用于生产者设置的最大请求大小，而服务端也会设置最大请求大小

- totalMemorySize - `buffer.memory`缓冲区大小，*默认32M*生产者可以用来缓冲等待发送到服务器的记录的总字节数。如果记录的发送速度比它们可以传递到服务器的速度快，生产者将阻塞 `MAX_BLOCK_MS_CONFIG`*（默认1min）*之后它会抛出异常，这个设置应该大致对应于生产者将使用的总内存，但不是硬限制，因为并非生产者使用的所有内存都用于缓冲。一些额外的内存将用于压缩（如果启用了压缩）以及用于维护进行中的请求。

- compressionType - `compression.type`配置生产者生成的所有数据的压缩类型。默认为none(即不压缩)。有效的值为`none`、`gzip`、`snappy`、`lz4`,压缩是整批数据，所以batching的效果也会影响压缩率（更多batching意味着更好的压缩）

  
