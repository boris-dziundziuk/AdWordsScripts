```js
var value_for_cpa = 30;
var value_for_cost = 40;
var campaign_contains = "магазин";

function main(){
  Logger.log(new Date());
  var table_one_or_more_than_one = SpreadsheetApp
                     .openById("1mW6wCSSwxbIifgPGTmmG6j3ZRcLW4EgW_NiLM-GepTE"); //ID таблицы Conversions>=1
  var table_less_than_one = SpreadsheetApp
                     .openById("1WxPELhwyiQj7L4WdeSxq3VynrpH0eaBRb-z3pymyDQc"); //ID таблицы Conversions<1
  
  var table_1 = convert_table(table_one_or_more_than_one,value_for_cpa);
  var table_2 = convert_table(table_less_than_one,value_for_cost);
  
  var MILLIS_PER_DAY = 1000 * 60 * 60 * 24;
  var now = new Date();
  var from = new Date(now.getTime() - 8 * MILLIS_PER_DAY);
  var to = new Date(now.getTime() - 1 * MILLIS_PER_DAY);
  var timeZone = AdWordsApp.currentAccount().getTimeZone();
  
  var query = 'SELECT CampaignName,AdGroupId,Id,Cost,CostPerConversion, Conversions ' +
    'FROM KEYWORDS_PERFORMANCE_REPORT WHERE CampaignName CONTAINS "'+campaign_contains+'" '+
    'AND Impressions>0 AND Status = "ENABLED" ' +
    'DURING ' + Utilities.formatDate(from, timeZone, 'yyyyMMdd') + ','
              + Utilities.formatDate(to, timeZone, 'yyyyMMdd');
  
  var arr_kw_ids = [];
  var report = AdWordsApp.report(query);
  var rows = report.rows();
  while(rows.hasNext()){
    var row = rows.next();
    //var kw_cn = row["CampaignName"];
    var kw_agid = row["AdGroupId"];
    var kw_id = row["Id"];    
    var kw_cost = row["Cost"];
    var kw_cpc = row["CostPerConversion"];
    var kw_conversions = row["Conversions"];
    var obj = new Object();
    obj.id = kw_id;
    obj.agid = kw_agid;
    //Logger.log("Conversion: " + kw_conversions);
    if(kw_conversions>=1){
      obj.label_position = position(kw_cpc, table_1);
      arr_kw_ids.push(obj);
      //Logger.log("Cpa: " + kw_cpc +"\t" + "Position: " + position(kw_cpc, table_1));
    }
    if(kw_conversions<1){
      obj.label_position = position(kw_cost, table_2);
      arr_kw_ids.push(obj);
      //Logger.log("Cost: " + kw_cost +"\t" + "Position: " + position(kw_cost, table_2));
    }
  }
    
    
  Logger.log(new Date());
 
  
  
  var s_array = separate_array(arr_kw_ids);
  
  for(var x = 0; x < s_array.length; x++){
    var ids = [];
    for(var ix = 0; ix < s_array[x].length; ix++){
      ids[ix] = [];
      ids[ix][0] = s_array[x][ix].agid;
      ids[ix][1] = s_array[x][ix].id;
      
    }
  	var kw_it = AdWordsApp.keywords().withIds(ids).get();
    while(kw_it.hasNext()){
      try{
      var kw = kw_it.next();
      kw.applyLabel(get_obj_by_id(s_array[x],kw.getId()).label_position);
      }catch(err){
        Logger.log(err);
      }
    }
  }  

  Logger.log(new Date());
}

function position(value, conver_table){
  for(var x = 0; x<conver_table.length; x++){
    var num1 = parseFloat(conver_table[x][0]);
    var num2 = parseFloat(conver_table[x][1]);
  	if(x==0 && value< num2){
      //Logger.log(value + "< " + conver_table[0][1]);
      return conver_table[x][2];
    }
    if(value>= num1 &&  value < num2 && x!=0){
      //Logger.log(value + ">=" + num1 + " && " + value + " < " + num2);
      return conver_table[x][2];
    }
    if(value>= num1 &&  x == conver_table.length-1 ){
      //Logger.log(value + ">=" + num1);
      return conver_table[x][2];
    }
  }
}

function convert_table(table_frome_gdrive,value){
  var range = table_frome_gdrive.getDataRange();
  var values = range.getValues();
  var conversion_table = []
  for(var index = 0; index < values.length; index++){
    conversion_table[index] = [];
    if(index == 0){
    	conversion_table[index][0] = 0;
    }else{
      conversion_table[index][0] = (values[index][0]*value).toFixed(2);
    }
    conversion_table[index][1] = (values[index][1]*value).toFixed(2);
    conversion_table[index][2] = values[index][2];
    //Logger.log(conversion_table[index][0]+"\t"+conversion_table[index][1]+"\t"+conversion_table[index][2]);
  }
  return conversion_table;
}


function separate_array(arr_kw_ids){
  var separate = [];
  var arr_n=0;  
  for(var s = 0; s<arr_kw_ids.length/10000; s++){
    separate[s] = [];
  	for(var num = 0;num<10000;num++,arr_n++){
      if(arr_n>=arr_kw_ids.length){
      	break;
      }
   	  separate[s].push(arr_kw_ids[arr_n]);
    }
  }
  return separate;
}

function get_obj_by_id(arr,id){
  for(var y = 0; y < arr.length; y++){
  	if(arr[y].id == id){
      return arr[y];
    }
  }
}
```
