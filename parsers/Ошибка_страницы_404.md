# Скрипт проверки объявлений с ошибкой 404
Скрипт перебирает все кампании и группы. В группах проверяются все обновления. Если объявление 
включено и скрипт получает 404 ошибку со страницы, то объявление выключается и наоборот. Если
объявление выключено и ошибка 404 не получена, то данное объявление включается. 
Настройка селекторов для выбора кампаний, групп:  
**https://github.com/promodo2018/AdWordsScripts/blob/master/TUTORIALS/Selectors.md**

```js
function main(){
  var params = {muteHttpExceptions:true, followRedirects: false};
  
  var camp_it = AdWordsApp.campaigns().get();
  
  while(camp_it.hasNext()){
  
  	var campaign = camp_it.next();
    var adGroup_it = campaign.adGroups().get();
    
    while(adGroup_it.hasNext()){
    
      var adGroup = adGroup_it.next();
      var ad_it = adGroup.ads().get();
      
      while(ad_it.hasNext()){
      
      	var ad = ad_it.next();
        var url = ad.urls().getFinalUrl();
        
        Logger.log("Cheking url - " + url);
        var resp = UrlFetchApp.fetch(url, params).getResponseCode();
        Logger.log(resp);
        if(resp == 404){
          ad.pause();
 //Logger.log(campaign.getName() + ": AdGroup " + adGroup.getName() +": ad " + ad.getId() + " - paused");
        }else{
          ad.enable();
 //Logger.log(campaign.getName() + ": AdGroup " + adGroup.getName() +": ad " + ad.getId() + " - enabled");
        }
      }
    }    
  }
}
```
