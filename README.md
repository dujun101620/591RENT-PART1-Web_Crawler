# 591租屋網 雙北「租屋補助」分析專案 Part1-Python程式爬蟲　　
  
## 專案背景
　　身為經營管理專員，我為了加強自己在數據分析領域的工具與技能，自學基礎的Python、MYSQL程式語言，以及Power BI視覺化工具。同時，身為社工師的親友經常需要協助案主找尋合適且適用「租屋補助」申請的租屋，並致力於相關社會議題的研討。因此藉此機會，我希望能運用所學的基礎能力，實作關於「租屋補助」的程式專案，**分析租屋市場樣貌，並在過程中學習解決程式問題並產出成果的能力，累積經驗與作品**。  
    
## 分析目的
　　為瞭解適用「租屋補助」之案件在租屋市場上與一般租屋之樣貌差異，截取2022年12月20日591租屋網上的台北市與新北市的租屋案件作為分析資料。後續按以下三種租屋類別進行分析：　　
1. **一般租屋**：包含全部資料，意為租屋市場整體樣貌。
2. **租補**：包含在「標題」與「屋主說」中有租補相關關鍵字之資料，意為能申請租補之案件的整體樣貌。
3. **租補且非社宅**：因租補與社會住宅屬性不同，但在租屋網上常有同時存在或混用的狀況，因此將社會住宅篩除，以「租補且非社宅」類別分析較純粹的租補樣貌。  
    
## 作法架構  
1. **資料取得與儲存：Python (本篇內容)**
2. 資料清理與處理：MYSQL
3. 視覺化圖表分析：Power BI

## 程式內容──資料取得與儲存：Python  
以台北市為例說明程式內容，新北市做法相同。  
### 一、爬取591租屋網台北市約12000~13000筆資料
#### 1. 匯入模組
```py
import urllib.request as req
import json
import pandas as pd
import bs4
```
  
#### 2. 創建自定義函數getData，目的在於截取每一筆案件頁面的多個欄位資訊
觀察後發現單筆案件的網址差異為結尾是發佈文章的號碼(ID)，此ID可在搜尋頁面的程式碼中找到。  
因此，步驟為取得搜尋頁面上的每筆案件ID並串成案件網址，再取得案件網頁的多個欄位資訊。  

##### (1) 爬取搜尋網頁，取得上面的每筆案件ID以形成案件網址
網頁使用AJAX技術、JASON語法、utf-8編碼。  
url：每頁搜尋頁面包含30筆租屋案件，且每頁搜尋頁面的網址差異在於結尾的firstRow=30*(頁數-1)。totalRows=總筆數，但測試後發現不影響結果，因此設定為最大資料筆數13000。  
request：須包含hearders為User-Agent、cookie、X-CSRF-TOKEN。  
ID資訊在data標籤→data標籤→post_id。  
將截取到的ID放入網址結尾以取得每筆案件的頁面網址。
```py
def getData(pages):
    result=list()
    for x in range(pages):
        y=30*x
        url="https://rent.591.com.tw/home/search/rsList?is_format_data=1&is_new_list=1&type=1&firstRow="+str(y)+"&totalRows=13000"
        request=req.Request(url,headers={
            "User-Agent":"輸入User-Agent",
            "cookie":"輸入cookie",
            "X-CSRF-TOKEN":"輸入X-CSRF-TOKEN"
        })
        with req.urlopen(request) as response:
            data=response.read().decode("utf-8")

        data=json.loads(data)
        posts=data["data"]["data"]

        for post in posts:
            ID=post["post_id"]
            URLS_ID=["https://bff.591.com.tw/v1/house/rent/detail?id="+str(ID)]
```

##### (2) 爬取單筆案件網頁的多個欄位資訊
request：須包含hearders為User-Agent、device、deviceid。  
觀察並截取所需欄位資料，共25欄。  
其中「屋主說」(result23)欄位使用html語法解讀避免亂碼。
```py
            for URL_ID in URLS_ID:
                request=req.Request(URL_ID,headers={
                    "User-Agent":"輸入User-Agent",
                    "device":"輸入device",
                    "deviceid":"輸入deviceid"
                })
                with req.urlopen(request) as response:
                    data=response.read().decode("utf-8")

                data=json.loads(data)
                try:
                    results1=data["data"]["breadcrumb"][0]["name"]
                except:
                    results1="無資料"
                    
                try:    
                    results2=data["data"]["title"]
                except:
                    results2="無資料"
                    
                try:
                    results3=data["data"]["favData"]["address"]
                except:
                    results3="無資料"
                    
                try:
                    results4=data["data"]["favData"]["area"]
                except:
                    results4="無資料"
                    
                try:
                    results5=data["data"]["favData"]["kindTxt"]
                except:
                    results5="無資料"
                    
                try:
                    results6=data["data"]["favData"]["price"]
                except:
                    results6="無資料"
                    
                try:
                    if data["data"]["costData"]["data"][0]["name"]=="租金含":
                        results7=data["data"]["costData"]["data"][0]["value"]
                    elif data["data"]["costData"]["data"][1]["name"]=="租金含":
                        results7=data["data"]["costData"]["data"][1]["value"]
                    elif data["data"]["costData"]["data"][2]["name"]=="租金含":
                        results7=data["data"]["costData"]["data"][2]["value"]
                    elif data["data"]["costData"]["data"][3]["name"]=="租金含":
                        results7=data["data"]["costData"]["data"][3]["value"]
                    else:
                        results7="無資料"
                except:
                    results7="無資料"
                    
                try:
                    if data["data"]["costData"]["data"][0]["name"]=="押金":
                        results8=data["data"]["costData"]["data"][0]["value"]
                    elif data["data"]["costData"]["data"][1]["name"]=="押金":
                        results8=data["data"]["costData"]["data"][1]["value"]
                    elif data["data"]["costData"]["data"][2]["name"]=="押金":
                        results8=data["data"]["costData"]["data"][2]["value"]
                    elif data["data"]["costData"]["data"][3]["name"]=="押金":
                        results8=data["data"]["costData"]["data"][3]["value"]
                    else:
                        results8="無資料"
                except:
                    results8="無資料"
                    
                try:
                    results9=data["data"]["info"][2]["value"]
                except:
                    results9="無資料"
                    
                try:
                    results10=data["data"]["info"][3]["value"]
                except:
                    results10="無資料"
                    
                try:
                    results11=data["data"]["positionRound"]["lat"]
                except:
                    results11="無資料"
                    
                try:
                    results12=data["data"]["positionRound"]["lng"] 
                except:
                    results12="無資料"
                    
                try:
                    results13=data["data"]["service"]["desc"]
                except:
                    results13="無資料"
                    
                try:
                    results14=data["data"]["service"]["rule"]
                except:
                    results14="無資料"
                    
                try:
                    if data["data"]["service"]["facility"][0]["active"]==0:
                        results15="無"+data["data"]["service"]["facility"][0]["name"]
                    elif data["data"]["service"]["facility"][0]["active"]==1:
                        results15="有"+data["data"]["service"]["facility"][0]["name"]
                    else:
                        results15="無資料"
                except:
                    results15="無資料"
                    
                try:
                    if data["data"]["service"]["facility"][1]["active"]==0:
                        results16="無"+data["data"]["service"]["facility"][1]["name"]
                    elif data["data"]["service"]["facility"][1]["active"]==1:
                        results16="有"+data["data"]["service"]["facility"][1]["name"]
                    else:
                        results16="無資料"
                except:
                    results16="無資料"
                    
                try:
                    if data["data"]["service"]["facility"][2]["active"]==0:
                        results17="無"+data["data"]["service"]["facility"][2]["name"]
                    elif data["data"]["service"]["facility"][2]["active"]==1:
                        results17="有"+data["data"]["service"]["facility"][2]["name"]
                    else:
                        results17="無資料"
                except:
                    results17="無資料"
                    
                try:
                    if data["data"]["service"]["facility"][3]["active"]==0:
                        results18="無"+data["data"]["service"]["facility"][3]["name"]
                    elif data["data"]["service"]["facility"][3]["active"]==1:
                        results18="有"+data["data"]["service"]["facility"][3]["name"]
                    else:
                        results18="無資料"
                except:
                    results18="無資料"
                    
                try:
                    if data["data"]["service"]["facility"][5]["active"]==0:
                        results19="無"+data["data"]["service"]["facility"][5]["name"]
                    elif data["data"]["service"]["facility"][5]["active"]==1:
                        results19="有"+data["data"]["service"]["facility"][5]["name"]
                    else:
                        results19="無資料"
                except:
                    results19="無資料"
                    
                try:
                    if data["data"]["service"]["facility"][6]["active"]==0:
                        results20="無"+data["data"]["service"]["facility"][6]["name"]
                    elif data["data"]["service"]["facility"][6]["active"]==1:
                        results20="有"+data["data"]["service"]["facility"][6]["name"]
                    else:
                        results20="無資料"
                except:
                    results20="無資料"
                    
                try:
                    if data["data"]["service"]["facility"][8]["active"]==0:
                        results21="無"+data["data"]["service"]["facility"][8]["name"]
                    elif data["data"]["service"]["facility"][8]["active"]==1:
                        results21="有"+data["data"]["service"]["facility"][8]["name"]
                    else:
                        results21="無資料"
                except:
                    results21="無資料"
                    
                try:
                    if data["data"]["service"]["facility"][13]["active"]==0:
                        results22="無"+data["data"]["service"]["facility"][13]["name"]
                    elif data["data"]["service"]["facility"][13]["active"]==1:
                        results22="有"+data["data"]["service"]["facility"][13]["name"]
                    else:
                        results22="無資料"
                except:
                    results22="無資料"
                    
                try:
                    results23=bs4.BeautifulSoup(data["data"]["remark"]["content"],"html.parser").get_text()                  
                except:
                    results23="無資料"

                Result=str(ID),results1,results2,results3,results4,results5,results6,results7,results8,results9,results10,results11,results12,results13,results14,results15,results16,results17,results18,results19,results20,results21,results22,results23,"https://rent.591.com.tw/home/"+str(ID)
                result.append(Result)
    return result
```

#### 3. 使用自定義函數getData
變數為頁數，共13000筆/每頁30筆，無條件進位=434。使搜尋頁面的434頁重複上述步驟取得所需資料。  
```py
DATAS=getData(434)
```
  
  
### 二、匯出csv檔
使用pandas DataFrame格式，編碼為utf_8_sig，將爬取資料輸出成csv檔。
```py
df=pd.DataFrame(DATAS,columns=["ID","地區","標題","地址","坪數","房型","價格","租金含","押金","樓層","建築種類","緯度","經度","租住說明","房屋守則","冰箱","洗衣機","電視","冷氣","床","衣櫃","網路","電梯","屋主說","網址"])
df.to_csv("591Taipei.csv",index=False,encoding="utf_8_sig")
```
  
### 三、匯至MYSQL
使用pymysql模組連接設立好的MYSQL資料庫(591rent)與表格(591rent_taipei)並匯入資料，資料庫使用utfmb4編碼避免亂碼。
```py
import pymysql

def save(Datas):
    db_settings={
        "host": "localhost",
        "port": 3306,
        "user": "root",
        "password": "密碼",
        "db": "591rent",
        "charset": "utf8mb4"
    }
    
    
    conn=pymysql.connect(**db_settings)
    with conn.cursor() as cursor:
        sql="""INSERT INTO `591rent_taipei`(`ID`,`地區`,`標題`,`地址`,`坪數`,`房型`,`價格`,`租金含`,`押金`,`樓層`,`建築種類`,`緯度`,`經度`,`租住說明`,`房屋守則`,`冰箱`,`洗衣機`,`電視`,`冷氣`,`床`,`衣櫃`,`網路`,`電梯`,`屋主說`,`網址`) VALUES(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s);"""
        for Data in Datas:
            cursor.execute(sql,Data)
        conn.commit()        
        cursor.close()
        conn.close()
    
save(DATAS)
```
  
### 四、回報完成
```py
print("finished")
```  

