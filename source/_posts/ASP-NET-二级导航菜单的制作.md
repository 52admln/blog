---
title: ASP.NET 二级导航菜单的制作
date: 2017-06-15 22:17:20
tags: ASP.NET, 二级导航
---

## HTML结构

```
<!--header end-->
        <!--sidebar start-->
        <aside>
            <div id="sidebar" class="nav-collapse">
                <!-- sidebar menu start-->
                <ul class="sidebar-menu" id="nav-accordion">
                    <li>
                        <a class="" href="index.html">
                            <i class="icon-home"></i>
                            <span>主页</span>
                        </a>
                    </li>
                    <li class="sub-menu">
                        <a class="active" href="javascript:;">
                            <i class="icon-archive"></i>
                            <span>商品管理</span>
                        </a>
                        <ul class="sub">
                            <li >
                                <a href="goods_list.html">商品列表</a>
                            </li>
                            <li class="active">
                                <a href="goods_group.html">商品类别</a>
                            </li>
                            <li>
                                <a href="goods_add.html">商品添加</a>
                            </li>
                        </ul>
                    </li>
                    <!-- ... -->
                </ul>
                <!-- sidebar menu end-->
            </div>
        </aside>
        <!--sidebar end-->
```
<!-- more -->
## 数据库设计

| menu_id   | menu_pid | menu_name|
| :-------- | --------:| :--------: |
| 1         | -1       |  一级导航1  |
| 2         | 1        |  二级导航1  |
| 3         | 1        |  二级导航2  |
| 4         | -1       |  一级导航2  |
| 5         | 4        |  二级导航2  |
| 6         | 4        |  二级导航2  |



>` menu_id`  为自增字段
> `menu_pid` 为二级、一级导航区分字段
> `menu_name` 为导航菜单名称

`menu_pid` 的值为  `-1` ，则代表当前记录为 一级菜单
`menu_pid` 的值为 一级菜单的ID的值，则这条记录属于该菜单的子导航



## 后台

### 一般处理程序
``` 
<%@ WebHandler Language="C#" Class="load_menu" %>

using System;
using System.Web;
using System.Data.OleDb;
using System.Data;
using System.Web.UI;
using System.Web.UI.HtmlControls;
using System.Web.UI.WebControls;
using System.Web.UI.WebControls.WebParts;
using System.Collections;
using System.Collections.Generic;
using System.Configuration;
using System.Linq;
using System.Web.Security;
using System.Xml.Linq;
using System.Web.Script.Serialization;
using System.Text;
using System.IO;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Formatters.Binary;

public class load_menu : IHttpHandler {
    
    public void ProcessRequest (HttpContext context) {
        //指定回送数据的类型
        context.Response.ContentType = "application/json";

        //准备回送的字符串
        string outstr = "";

        //执行写入操作
        outstr = doSelect();

        //回送结果
        context.Response.Write(outstr);
    }

    public string doSelect()
    {
        //执行查询操作，并获得返回的行数
        DataSet ds = DBHelper.sqlMethod("select menu_id, menu_pid, menu_name from t_treemenu order by menu_id asc");
        int i = ds.Tables[0].Rows.Count;

        StringBuilder Builder = new StringBuilder();
        Builder.Append("{");
        Builder.Append("\"r\":" + i + "");

        if (i > 0)
        {
            List<MyMenu> list = new List<MyMenu>();

            for (int j = 0; j < i; j++)
            {
                MyMenu temp = new MyMenu(int.Parse(ds.Tables[0].Rows[j][0].ToString()), int.Parse(ds.Tables[0].Rows[j][1].ToString()), ds.Tables[0].Rows[j][2].ToString());
                list.Add(temp);
            }

            string json = new JavaScriptSerializer().Serialize(list);//这个很关键，否则error 

            Builder.Append(",\"data\":");
            Builder.Append(json);
        }

        Builder.Append("}");
        return Builder.ToString();
    }
    
    public bool IsReusable {
        get {
            return false;
        }
    }

}
```
### 类

```
using System;
using System.Data;
using System.Configuration;
using System.Web;
using System.Web.Security;
using System.Web.UI;
using System.Web.UI.WebControls;
using System.Web.UI.WebControls.WebParts;
using System.Web.UI.HtmlControls;

/// <summary>
/// MyMenu 的摘要说明
/// </summary>
public class MyMenu
{
    public int id;
    public int pid;
    public string name;

	public MyMenu()
	{
		//
		// TODO: 在此处添加构造函数逻辑
		//
	}
    public MyMenu(int id, int pid, string name)
    {
        this.id = id;
        this.pid = pid;
        this.name = name;
    }
    public int getId()
    {
        return this.id;
    }

    public int getPid()
    {
        return this.pid;
    }

    public string getName()
    {
        return this.name;
    }
}

```

## 前台渲染

```
// tree menu


 function loadMenu()
{
  var request = $.ajax({  
      url: "load_menu.ashx",  
      type: "POST",  
      async: false,  
      dataType: "json",  
      cache: false,  
      
	  // 返回结果成功后的回调
      success: function (data) {  
          var json=data;
          console.log(json);
          renderMenu(json.data);

      },  
      error: function (XMLHttpRequest, textStatus, errorThrown) { console.log(XMLHttpRequest); }  
  }); 
}

function renderMenu(data)
{
  // HTML代码
  var mystr="";
  var myData = data;
  $.each(data, function (index, item) {
	  // 一级导航的 ID 值
      var curId = item.id;
      //循环获取数据  
      mystr += '<li class="sub-menu">'
      // 如果当前的 PID 等于-1，代表为一级导航
      if(item.pid == "-1") {
      console.log(data,index);
         mystr += 
              '<a href=\"\">'+
                 ' <i class="icon-archive"></i>'+
                 ' <span>'+item.name+'</span>'+
             ' </a>';
               mystr += 
              '<ul class="sub">';
             $.each(myData, function(index, item) {
                  // 如果当前的 PID 等于一级导航的ID值，也就是为一级导航的二级导航
                  if(item.pid == curId) {
                  console.log(item.name);
                       mystr += '<li><a href="goods_list.html">'+item.name+'</a></li>';
                  }
             })
             mystr += '</ul>';  
      }
      mystr += '</li>';
  });

  console.log(mystr);
  $("#nav-accordion").html(mystr);
}


loadMenu();
```


