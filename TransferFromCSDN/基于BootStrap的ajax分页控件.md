# 基于 BootStrap 的 ajax 分页控件
---

## 一、引入js文件和css文件
``` javascript
/*
 * 基于bootstrap的分页控件
 * 使用前请先引入：
 * 1.<link rel="stylesheet" href="bootstrap-3.3.0/dist/css/bootstrap.css" />
 * 2.<script type="text/javascript" src="jquery/jquery-1.11.2.min.js"></script>
 * 
 * 
 url        请求的url（目前没有，可用于拓展ajax的部分）
 pageSize   每页显示的数据量
 pageNo     当前页的序号
 totalSize  记录的总条数
 pageStyle  分页样式，支持bootstrap分页样式
 pageForm   查询表单
 pageDiv    展示分页控件的容器
 fn         点击分页按钮时触发的函数，类似于回调函数
 */

function Page(){
    this.url="";
    this.pageSize=10;
    this.pageNo=1;
    this.totalSize="";
    this.pageStyle="pagination";
    this.pageForm="";
    this.pageDiv="";
    this.fn="";
    this.setUrl=function(url){
        this.url=url;
    }
    this.getUrl=function(){
        return this.url;
    }
    this.setPageSize=function(pageSize){
        this.pageSize=Number(pageSize);
    }
    this.getPageSize=function(){
        return this.pageSize;
    }
    this.setPageNo=function(pageNo){
        this.pageNo=Number(pageNo);
    }
    this.getPageNo=function(){
        return this.pageNo;
    }
    this.setTotalSize=function(totalSize){
        this.totalSize=Number(totalSize);
    }
    this.getTotalSize=function(){
        return this.totalSize;
    }
    this.setPageStyle=function(pageStyle){
        this.pageStyle=pageStyle;
    }
    this.getPageStyle=function(){
        return this.pageStyle;
    }
    this.setPageForm=function(pageForm){
        this.pageForm=pageForm;
    }
    this.getPageForm=function(){
        return this.pageForm;
    }
    this.setPageDiv=function(pageDiv){
        this.pageDiv=pageDiv;
    }
    this.getPageDiv=function(){
        return this.pageDiv;
    }
    this.setFn=function(fn){
        this.fn=fn;
    }
    this.getFn=function(){
        return this.fn;
    }
    
    
    this.doStartPage=function(){
        //判断是否是查询分页还是直接分页
        var html=formGeneratePage(this);
        console.log(html);
        $("#"+this.pageDiv+"").empty();
        $("#"+this.pageDiv+"").append(html);
    }
    
    /**
     * 封装的一个小工具，获取表单所有样式为 form-data 的 input
     */
    this.getFormData=function(pageFrom){
        this.pageForm=pageFrom;
        var data="";
        $("#"+this.getPageForm()+" input.form-data").each(function(count){
            var name=$(this).attr("name");
            var value=$(this).attr("value");
            if(count!=0){
                data=data+"&";
            }
            data=data+name+"="+value;
        });
        return data;
    }
    
}


/**
 * 支持表单查询的分页
 */
function formGeneratePage(bean){
    //计算总页数
    var pageCount=parseInt((bean.getTotalSize()+bean.getPageSize()-1)/bean.getPageSize());
    
    var html="";
    html=html+"<style type='text/css'>";
    html=html+".pointer{cursor: pointer; }";
    html=html+"</style>";
    html=html+"<ul class='" + bean.getPageStyle() + "'>";
    
    if(bean.getTotalSize()==0){//没有记录
        html=html+"<strong>没有记录</strong>";
    }else{// 有记录
        // 1.不需要分页，记录条数小于pageSize
        // 2.当前页是第一页，不需要显示上一页
        // 3.最后一页，不需要显示下一页
        if(bean.getPageNo()==1){
            html=html+"<li class='disabled'><a class='pointer'  onclick='return false'>上一页</a></li>";
        }else{
            html=html+"<li><a class='pointer' onclick='javascript:"+bean.getFn()+"(\""+(bean.getPageNo() - 1)+"\");'>";
            html=html+"上一页</a></li>";
        }
        
        //如果页数过多  
        var start=1;
        if(bean.getPageNo()>4){
            start=bean.getPageNo()-1;
            html=html+"<li><a class='pointer' onclick='javascript:"+bean.getFn()+"(\"1\");'>";
            html=html+"1</a></li>";
            html=html+"<li><a class='pointer' onclick='javascript:"+bean.getFn()+"(\"2\");'>";
            html=html+"2</a></li>";
            html=html+"<li><a class='pointer'  onclick='return false'>…</a></li>";
        }
        //显示当前页相邻的页 
        var end=bean.getPageNo()+1;
        if(end>pageCount){
            end=pageCount;
        }
        for(var i=start;i<=end;i++){
            if(bean.getPageNo()==i){
                html=html+"<li class='active'><a class='pointer'  onclick='return false'>" + bean.getPageNo()+ "</a></li></li>";
            }else{
                html=html+"<li><a class='pointer' onclick='javascript:"+bean.getFn()+"(\""+i+"\");'>";
                html=html+"" + i + "</a></li>";
            }
        }
        
        //后面的页数过多 html=html+a;
        if(end<pageCount-2){
            html=html+"<li><a class='pointer'  onclick='return false'>…</a></li>";
        }
        
        if(end<pageCount-1){
            html=html+"<li><a class='pointer' onclick='javascript:"+bean.getFn()+"(\""+(pageCount - 1)+"\");'>";
            html=html+(pageCount - 1) + "</a></li>";
        }
        if(end<pageCount){
            html=html+"<li><a class='pointer' onclick='javascript:"+bean.getFn()+"(\""+pageCount+"\");'>";
            html=html+pageCount + "</a></li>";
        }
        if(bean.getPageNo()==pageCount){
            html=html+"<li class='disabled'><a class='pointer' onclick='return false'>下一页</a></li>";
        }else{
            html=html+"<li><a class='pointer' onclick='javascript:"+bean.getFn()+"(\""+(bean.getPageNo() + 1)+"\");'>";
            html=html+"下一页</a></li>";
        }
    }
    html=html+"</ul>";
    return html;
}

```

## 二、使用方式
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
        <link rel="stylesheet" href="bootstrap-3.3.0/dist/css/bootstrap.css" />
        <style type="text/css"></style>
    </head>
    <body>
        <!-- 用于展示分页控件的容器 -->
        <div id="pageDiv"></div>
        <div class="clearfix"></div>
        
        <form id="showData" class="navbar-form navbar-left" role="search">
            <div class="form-group">
                <input type="text" name="username" value="Neo" class="form-control form-data" />
            </div>
            <div class="form-group">
                <input type="password" name="password" value="admin" class="form-control form-data" />
            </div>
            <button type="button" class="btn btn-default" onclick="getFormData()">Submit</button>
        </form>

    </body>
    <script type="text/javascript" src="jquery/jquery-1.11.2.min.js"></script>
    <script type="text/javascript" src="js/paging.js"></script>
    <script type="text/javascript">
        function showPage(pageNo) {
            var totalSize = 100;
            if(typeof(pageNo) == "undefined") {
                pageNo = 1;
            }
            var bean = new Page();
            bean.setPageNo(pageNo);//设置当前页
            bean.setTotalSize(totalSize);//设置总页数
            bean.setPageDiv("pageDiv");//设置展示分页控件的容器
            bean.setFn("showPage");//点击按钮的回调函数
            bean.doStartPage();
        }
        showPage(1)

        function getFormData() {
            var bean = new Page();
            var data = bean.getFormData("showData")
            alert(data);
        }
    </script>

</html>
```
> 本文章迁移于CSDN[基于 BootStrap 的 ajax 分页控件](https://blog.csdn.net/after95/article/details/52885659)