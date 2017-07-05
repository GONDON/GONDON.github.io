---
title: 仿z-tree作数据连接渲染
date: 2017-07-04 21:10:34
categories: Testing
---
### 模糊搜索
```
var data={
	 	getNodesByParamFuzzy: function (setting, nodes, key, value) {
	                if (!nodes || !key) return [];
	                var childKey = setting.data.key.children,
	                    result = [];
	                value = value.toLowerCase();
	                for (var i = 0, l = nodes.length; i < l; i++) {
	                    if (typeof nodes[i][key] == "string" && nodes[i][key] .toLowerCase().indexOf(value) > -1) {	//用字符串indexof来做匹配，还可以用正则
	                        result.push(nodes[i]);
	                    }
	                    result = result.concat(data.getNodesByParamFuzzy(setting, nodes[i][childKey], key, value));
	                }
	                return result;
	            }
 }
var getNodesByParamFuzzy = function (key, value, parentNode) {
        if (!key) return null;
        return data.getNodesByParamFuzzy(setting, parentNode ? parentNode[setting.data.key.children] : data.getNodes(setting), key, value);
    }

    $("search").on("input propertychange",function(){
	    	var value = $(this).val();
	    	var data = dept.getNodesByParamFuzzy("name",value);
	    	if(data != null){

	    	}
    	})
```