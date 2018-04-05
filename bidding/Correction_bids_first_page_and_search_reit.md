

```js
function main(){ 
  
  var maxBidCpcForFirstPage = 1; // Максимальная ставка
  var searshReiting = 80; // Проценты. От 0 до 100
  var labelName = "testLabel"; // Ввести имя ярлыка, которым помечены ключевые слова
  var labels = [];
  var kw_it = AdWordsApp.labels().withCondition("Name = '"+ labelName +"'").get();
  while(kw_it.hasNext()){
    var kw = kw_it.next();
    labels.push(kw.getId());
  }  
  if(labels.length == 0){
    Logger.log("А аккаунте нет ярлыка - " + labelName);
  }  
  var query = "SELECT Id,AdGroupId, FirstPageCpc,CpcBid,SearchImpressionShare "+ 
  "FROM KEYWORDS_PERFORMANCE_REPORT " +
  "WHERE LabelIds CONTAINS_ANY [" + labels.join(',')+"] DURING TODAY"; // Данные за текущий день

  var report = AdWordsApp.report(query);
  var rows = report.rows();
  
  var keywords = [];
  var keywordIds = [];
  while(rows.hasNext()){
    var row = rows.next();
    var SIS = parseFloat(row['SearchImpressionShare'].replace(/</g,""));
    if(isNaN(SIS)){
      SIS = 0;
    }
    if(SIS < searshReiting ){  // Сравнение с необходимым рейтингом
      var obj = new Object();
      var FPC = parseFloat(row['FirstPageCpc']);
      var CPC = parseFloat(row['CpcBid']);
      obj.id = parseFloat(row["Id"]);
      if(CPC < maxBidCpcForFirstPage && CPC <= FPC){ 
        // Если ставка ключевого слова меньше максимальной и меньше либо
        //равна ставке для первой страницы, ставка повышается на 0.01
        obj.cpc = FPC + 0.01;
      }
      if(CPC >= maxBidCpcForFirstPage){  
        // Если ставка больше либо равна максимальному установленному значению, 
        //ставка приравнивается к максимальному значению
        obj.cpc = maxBidCpcForFirstPage;
      }      
      keywords.push(obj);
      keywordIds.push([row["AdGroupId"],row["Id"]]);
    }
  }

  var keyword_it = AdWordsApp.keywords().withIds(keywordIds).get();
  while(keyword_it.hasNext()){
    var keyword = keyword_it.next();
    keyword.bidding().setCpc(cpc(keywords,keyword.getId()));
  }
}

function cpc(keywords, keywordId) {
  for(var index = 0; index<keywords.length; index++){
    if(keywordId == keywords[index].id){
      return keywords[index].cpc;
    }
  }
}
```
