# Остановка или запуск групп в зависимости от наличия активных акций на сайте

Для работы необходимо сформировать фид с id групп и ссылками для парсинга.
В 23 строке необходимо вставить элемент, по которому определяется отсутствие
активных акций, если данный элемент на не найден - группа запускается.

Полезно(помощь для вставки элемента): регулярные выражения в js
https://github.com/promodo2018/AdWordsScripts/blob/master/TUTORIALS/Regular_expressions.md

```js
function main() {
  var data = SpreadsheetApp.openByUrl(
  "https://docs.google.com/spreadsheets/d/1VtM8luZT8eb_joaZVy1SBPbxOr8kRttwdsQKo34y69o/edit#gid=0");//URL фида
  var params = {muteHttpExceptions:true, followRedirects: false};
  var range = data.getDataRange();
  var values = range.getValues();
  var adGroupIdsEnabled = [];
  var adGroupIdsPaused = [];
  for(var index = 1; index < values.length; index++){
    try{
      var url = values[index][1];
      var html_page = UrlFetchApp.fetch(url,params).getContentText();
      var cheking = html_page.match(/<div class="campaign small finished">/g || []);
      //Вставить элемент межу /элемент/ для определения активных акций
      if(cheking == null){
        adGroupIdsEnabled.push(parseFloat(values[index][0]));
      }else{
        adGroupIdsPaused.push(values[index][0]);
      }
    }catch(err){
Logger.log(err + " - " +values[index][0]+" "+values[index][1]+" "+values[index][2]+" "+values[index][3]);
    }
  }
  var adGroupEnabled_it = AdWordsApp.adGroups().withIds(adGroupIdsEnabled).get();
  var adGroupPaused_it = AdWordsApp.adGroups().withIds(adGroupIdsPaused).get();
  var enable = 0;
  var pause = 0;
  while(adGroupEnabled_it.hasNext()){
    var adGroup = adGroupEnabled_it.next();
    if(adGroup.isPaused()){
      adGroup.enable();
      enable++;
    }
  }
  while(adGroupPaused_it.hasNext()){
    var adGroup = adGroupPaused_it.next();
    if(adGroup.isEnabled()){
      adGroup.pause();
      pause++;
    }
  }
  Logger.log("Enabled - " + enable);
  Logger.log("Paused - " + pause);
}
```
