# kinjiSpider

## 運作時間
大約5~6分鐘

## 套件需求

```
from bs4 import BeautifulSoup
import requests
import sqlite3
import aiohttp
import asyncio
import time
```

## 簡介

### 取得所有要爬的網址

```
listBook = []
response = requests.get('http://kanji.zinbun.kyoto-u.ac.jp/db-machine/ShikoTeiyo/')
response.encoding = 'utf-8'
soup = BeautifulSoup(response.text, 'lxml')
hrefAll = soup.find_all('a', href=True)
for i in hrefAll:
    url = i.get('href')
    listBook.append(f'http://kanji.zinbun.kyoto-u.ac.jp/db-machine/ShikoTeiyo/{url}')
```

### 啟動非同步處理

```
async def main():
    task = []
    sem = asyncio.Semaphore(10)
    for i in listBook:
        task.append(asyncio.create_task(
            SuperGiraffe(url=i, sem=sem)
        ))
    await asyncio.gather(*task)
    end = time.time()
    print(f'{round(end-start, 3)}s')
```

### 非同步處理函式

```
async def SuperGiraffe(url, sem):
    async with sem:
        async with aiohttp.ClientSession() as response:
            responseBody = await response.request('GET', url=url)
            soup = BeautifulSoup(await responseBody.text(), 'lxml')
            await response.close()
            if soup.find('font') == None:
                pass
            elif len(soup.find('font').text) >= 2:
                bookName = soup.find('font').text.replace('\n', '')
                #print(bookName)
                #print(f'CREATE TABLE IF NOT EXISTS {bookName}(chapter TEXT, content TEXT, url TEXT)')
                sqlite.execute(f'CREATE TABLE IF NOT EXISTS {bookName}(chapter TEXT, content TEXT, url TEXT)')
                contentMain = soup.find('p').text
                chapter = soup.find('h2').text.replace('\n', '')
                print(f'book:{bookName}, chapter:{chapter} parse')
                if soup.find('blockquote') != None:
                    contentExtra = soup.find('blockquote').text
                else:
                    contentExtra = ''
                content = contentMain + contentExtra
                sqlite.execute(f'INSERT INTO {bookName} (`chapter`, `content`, `url`) VALUES (\'{chapter}\', \'{content}\', \'{url}\')')
                print(f'book:{bookName}, chapter:{chapter} done')
                conn.commit()
```
