---
title: nexacro_align.md
categories:
  - source
tags:
  - nexacro
date: 2020-09-06 14:20:34
---

넥사크로 그리드 헤더 클릭시 정렬하는 소스

최근 프로젝트에서 모던JS와는 만리장성을 쌓은 넥사크로툴을 사용중인데 그리드 헤더 클릭시 정렬하는 부분을 직접 만들어야 한다고 해서 소스를 만들었다.

{% codeblock %}
this.executeGridSort = function (obj,e) 
{
	var ARROW_ASC = "↑";
	var ARROW_DESC = "↓";
	var sortStr = "S:";
	
	if(!obj.sortObj) obj.sortObj = new Object();
	var bindDs = obj.getBindDataset();
	var targetID = bindDs.getColumnInfo(e.cell).id;
	
	if(obj.sortObj[targetID]) {
		if(obj.sortObj[targetID] === ARROW_ASC) {
			obj.sortObj[targetID] = ARROW_DESC;
			obj.setCellProperty("head", e.cell, "text", obj.getCellText(-1, e.cell).substr(0,obj.getCellText(-1, e.cell).length-1)+ARROW_DESC);
		} else if (obj.sortObj[targetID] === ARROW_DESC) {
			delete obj.sortObj[targetID];
			obj.setCellProperty("head", e.cell, "text", obj.getCellText(-1, e.cell).substr(0,obj.getCellText(-1, e.cell).length-1));
		}
	} else {
		obj.sortObj[targetID] = ARROW_ASC;
		obj.setCellProperty("head", e.cell, "text", obj.getCellText(-1, e.cell)+ARROW_ASC);
	}
	
	for(var prop in obj.sortObj) {
		if(obj.sortObj.hasOwnProperty(prop)) {
			if(obj.sortObj[prop] == ARROW_ASC) {
				sortStr += "+" + prop;
			} else if (obj.sortObj[prop] == ARROW_DESC) {
				sortStr += "-" + prop;
			}
		}
	}
	
	if(sortStr !== "S:") {
		bindDs.set_keystring(sortStr);
	}
}
{% endcodeblock %}

위의 코드는 직접 만들었고 그리드에서 onheaderclick 함수를 만들어서 함수에서 주어지는 파라미터인 obj, e를 그대로 넣어서 던지면 된다.

다른 방법도 있을것 같지만 이 유사 JS개발툴은 뭘 알려주질 않아서 모르겠다.