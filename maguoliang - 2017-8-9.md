# 四川高院绩效考核系统代码复查  

## 具体问题

### 1、双重循环   

- 方法：`public boolean xzTsgk(JSONArray ajarray)`

- 代码：

  ```java
  @Override
  public void xzTsgk(JSONArray ajarray) {
    List<TSpsjAjXzsfgk> TSpsjAjXzsfgkList = new ArrayList<TSpsjAjXzsfgk>();
    //根据案件编号List查询所有评查案件相关的案件信息
    List<TSpsjAjUnionall> ajList = tjajService.getAjList(ajarray);
    try {
      Iterator<JSONObject> ajiter = ajarray.iterator();
      while (ajiter.hasNext()) {
        JSONObject temp = ajiter.next();
        TSpsjAjXzsfgk tsaj = new TSpsjAjXzsfgk();
        TSpsjAjUnionall aj = new TSpsjAjUnionall();
        for (TSpsjAjUnionall ajTemp : ajList) {
          if (ajTemp.getCAh().equals((temp).get("ah"))) {
            aj = ajTemp;
            break;
          }
        }
        setTsgkObject(tsaj, temp, aj);
        TSpsjAjXzsfgkList.add(tsaj);
      }
      tSpsjAjXzsfgkService.insertTSpsjAjXzsfgkList(TSpsjAjXzsfgkList);
    } catch (Exception e) {
      logger.error("添加庭审公开信息失败", e);
    }
  }  
  ```

- 问题：嵌套循环,while循环中嵌套了for循环，改进办法是把for循环方法取值更改，修改成`Map<key,value>`

- 改进：

  ```java
  @SuppressWarnings("unchecked")
  @Override
  public void xzTsgk(JSONArray ajarray) {
    List<TSpsjAjXzsfgk> TSpsjAjXzsfgkList = new ArrayList<TSpsjAjXzsfgk>();
    //根据案件编号List查询所有评查案件相关的案件信息
    Map<Long, TSpsjAjUnionall> ajList = tjajService.getAjList(ajarray);
    try {
      Iterator<JSONObject> ajiter = ajarray.iterator();
      while (ajiter.hasNext()) {
        JSONObject temp = ajiter.next();
        TSpsjAjXzsfgk tsaj = new TSpsjAjXzsfgk();
        TSpsjAjUnionall aj = ajList.get(temp.get("ajbh"));
        setTsgkObject(tsaj, temp, aj);
        TSpsjAjXzsfgkList.add(tsaj);
      }
      tSpsjAjXzsfgkService.insertTSpsjAjXzsfgkList(TSpsjAjXzsfgkList);
    } catch (Exception e) {
      logger.error("添加庭审公开信息失败", e);
    }
  }
  ```

- 代码片段2：

  ```java
  @SuppressWarnings("unchecked")
  public Map<Long, TSpsjAjUnionall> getAjList(JSONArray array) {
    //遍历案件并把案件编号储存
    Iterator<JSONObject> ajiterClah = array.iterator();
    List<Long> ajbhs = new ArrayList<>();
    while (ajiterClah.hasNext()) {
      JSONObject temp = ajiterClah.next();
      if (null != temp.get("ajbh")) {
        Long ajbh = Long.parseLong(String.valueOf(temp.get("ajbh")));
        ajbhs.add(ajbh);
      }
    }
    //根据案件编号List查询所有评查案件相关的案件信息
    Map<Long, TSpsjAjUnionall> ajList = ajUnionallMapper.getAjListByAjbhs(ajbhs);
    return ajList;
  }  
  ```

- dao层代码：

  ```java
  @MapKey("NBh")
  Map<Long, TSpsjAjUnionall> getAjListByAjbhs(List<Long> ajbhs);
  ```

  ​

- 提出人：马国梁

- 提出时间：2018-8-11

### 2、JS方法未返回值，导致页面缓存数据未清理

- 方法：`getJson()`

- 代码：  

  ```javascript
  /**
   * 获取页面缓存数据
   * @returns
   */
  function getJson() {
  	var zbid = $("#jqSelectList5bc7a_hidden").val();
  	if (zbid == null || zbid == "") {
  		Artery.showMessage("请选择审判数据补充类型！");
  		return;
  	}
  	if (zbid == 500101 || zbid == 500102 || zbid == 500109) {
  		var parentDiv = $("#jqGrid97394 > div[class=jqGrid-container]").find(
  				"table > tbody > tr");
  	} else if (zbid == 500121) {
  		var parentDiv = $("#jqGrid64d6d > div[class=jqGrid-container]").find(
  				"table > tbody > tr");
  	} else if (zbid == 600003) {
  		var parentDiv = $("#jqGrid83829 > div[class=jqGrid-container]").find(
  				"table > tbody > tr");
  	} else {
  		var parentDiv = $("#jqGrid740be > div[class=jqGrid-container]").find(
  				"table > tbody > tr");
  	}
  	var ajLength = parentDiv.length;
  	if (ajLength > 0) {
  		var result = new Array();
  		parentDiv.each(function(e) {
  			var obj = {};
  			var ah = $(this).find("td")[0].innerHTML;
  			var cbr = $(this).find("[jg-td-data]").attr("jg-td-data");
  			var cbrBh = eval('(' + cbr + ')').value;
  			var ajbh = parentDiv[e].id;
  			if ($(this).find("div[id^=jqValue] > input").length > 1) {
  				var value1 = $(this).find("div[id^=jqValue] > input")[1].value;
  				obj["value1"] = value1;
  			}
  			obj["ah"] = ah;
  			obj["cbr"] = cbrBh;
  			obj["ajbh"] = parseInt(ajbh);
  			result.push(obj);
  		});
  		var ajJson = JSON.stringify(result);
  		return ajJson;
  	}
  }
  ```


  ```java
  /**
       * 用于审判数据补充新增案件回显
       * @return
       */
  public List<Map<String, Object>> getRs(){
    //新添加的案件信息
    String ajbhs = ArteryParamUtil.getString("ajbhS");
    if(StringUtils.isBlank(ajbhs)){
      return new ArrayList<>();
    }
    String[] ajbh = ajbhs.split(",");
    //页面缓存的案件信息
    String ajs = ArteryParamUtil.getString("ajJson");
    com.alibaba.fastjson.JSONArray aj = com.alibaba.fastjson.JSONArray.parseArray(ajs);
    List<Map<String, Object>> res = new ArrayList<>();
    if (StringUtils.isNotBlank(ajs)) {
      for (int i = 0; i < aj.size(); i++) {
        JSONObject temp = aj.getJSONObject(i);
        res.add(temp);
      }
    }
    for (int j = 0; j < ajbh.length; j++) {
      if(ajbh[j]!=null){
        Map<String, Object> mapTemp = new HashMap<>();
        String[] temp = ajbh[j].split(";");
        if(StringUtils.isNotBlank(ajs)&& aj.toString().contains(temp[0])){
          continue;
        }
        if (temp.length < 2) {
          mapTemp.put("bh", ArteryUtil.getUuid());
          mapTemp.put("ah", temp[0]);
        } else {
          mapTemp.put("bh", ArteryUtil.getUuid());
          String ajbs = temp[2];
          mapTemp.put("ah", temp[0]);
          mapTemp.put("cbr", temp[1]);
          mapTemp.put("ajbh", ajbs);
        }
        res.add(mapTemp);
      }
    }
    return res;
  }
  ```

  ​

- 问题：1.js未返回值，导致session数据未及时清理；2.魔法数字

- 改进：

  ```javascript
  /**
   * 获取页面缓存数据
   * @returns
   */
  function getJson() {
  	var zbid = $("#jqSelectList5bc7a_hidden").val();
  	if (zbid == null || zbid == "") {
  		Artery.showMessage("请选择审判数据补充类型！");
  		return;
  	}
  	if (zbid == 500101 || zbid == 500102 || zbid == 500109) {
  		var parentDiv = $("#jqGrid97394 > div[class=jqGrid-container]").find(
  				"table > tbody > tr");
  	} else if (zbid == 500121) {
  		var parentDiv = $("#jqGrid64d6d > div[class=jqGrid-container]").find(
  				"table > tbody > tr");
  	} else if (zbid == 600003) {
  		var parentDiv = $("#jqGrid83829 > div[class=jqGrid-container]").find(
  				"table > tbody > tr");
  	} else {
  		var parentDiv = $("#jqGrid740be > div[class=jqGrid-container]").find(
  				"table > tbody > tr");
  	}
  	var ajLength = parentDiv.length;
  	if (ajLength > 0) {
  		var result = new Array();
  		parentDiv.each(function(e) {
  			var obj = {};
  			var ah = $(this).find("td")[0].innerHTML;
  			var cbr = $(this).find("[jg-td-data]").attr("jg-td-data");
  			var cbrBh = eval('(' + cbr + ')').value;
  			var ajbh = parentDiv[e].id;
  			if ($(this).find("div[id^=jqValue] > input").length > 1) {
  				var value1 = $(this).find("div[id^=jqValue] > input")[1].value;
  				obj["value1"] = value1;
  			}
  			obj["ah"] = ah;
  			obj["cbr"] = cbrBh;
  			obj["ajbh"] = parseInt(ajbh);
  			result.push(obj);
  		});
  		var ajJson = JSON.stringify(result);
  		return ajJson;
  	}else{
  		return null;
  	}
  }
  ```

  ```java
  /**
       * 用于审判数据补充新增案件回显
       * @return
       */
  public List<Map<String, Object>> getRs(){
    //新添加的案件信息
    String ajbhs = ArteryParamUtil.getString("ajbhS");
    if(StringUtils.isBlank(ajbhs)){
      return new ArrayList<>();
    }
    String[] ajbh = ajbhs.split(",");
    //页面缓存的案件信息
    String ajs = ArteryParamUtil.getString("ajJson");
    com.alibaba.fastjson.JSONArray aj = com.alibaba.fastjson.JSONArray.parseArray(ajs);
    List<Map<String, Object>> res = new ArrayList<>();
    if (StringUtils.isNotBlank(ajs)) {
      for (int i = 0; i < aj.size(); i++) {
        JSONObject temp = aj.getJSONObject(i);
        res.add(temp);
      }
    }
    for (int j = 0; j < ajbh.length; j++) {
      if(ajbh[j]!=null){
        Map<String, Object> mapTemp = new HashMap<>();
        String[] temp = ajbh[j].split(";");
        if(StringUtils.isNotBlank(ajs)&& aj.toString().contains(temp[NO_AH])){
          continue;
        }
        if (temp.length < NO_WCBR) {
          mapTemp.put("bh", ArteryUtil.getUuid());
          mapTemp.put("ah", temp[NO_AH]);
        } else {
          mapTemp.put("bh", ArteryUtil.getUuid());
          mapTemp.put("ah", temp[NO_AH]);
          mapTemp.put("cbr", temp[NO_CBR]);
          mapTemp.put("ajbh", temp[NO_AJBS]);
        }
        res.add(mapTemp);
      }
    }
    return res;
  }
  ```

  ​

- 提出人：马梁

- 提出时间：2017-8-16