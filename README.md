# ChatGPT を使った Webスクレイピング

[![Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/)

このガイドでは、ChatGPT を使用して、静的および複雑な Webサイトの両方を対象に、Webスクレイピング用の Python スクリプトを生成する方法を説明します。

- [前提条件](#prerequisites)  
- [ChatGPT を使って静的 HTML の Webサイトをスクレイピングする](#scraping-websites-with-static-html-using-chatgpt)  
- [複雑な Webサイトのスクレイピング](#scraping-complex-websites)  

## Prerequisites

このガイドに沿って進めるには、Python に慣れていてローカルにインストールされていること、また ChatGPT アカウントをお持ちであることが必要です。

ChatGPT を使って Webスクレイピングスクリプトを生成する際には、主に 2 つのステップがあります。

ChatGPT を使用して Webスクレイピングスクリプトを作成するには、次の手順に従ってください。

1. **ターゲット要素を特定する**  
   - データを見つけて抽出するために必要な手順を文書化します。  
   - 目的のページ要素を右クリック → **Inspect** → **Copy > Copy selector** を選択して HTML パスを取得します。  

   ![Copying a selector](https://github.com/bright-jp/web-scraping-with-chatgpt/blob/main/images/Copying-a-selector-1.png)

2. **ChatGPT でコードを生成する**  
   - スクレイピングのロジックを指定する、明確で詳細なプロンプトを提示します。

3. **実行とテスト**  
   - 生成されたスクリプトを実行し、結果を検証します。

## Scraping Websites with Static HTML Using ChatGPT

ChatGPT を使って、[static HTML](https://www.w3schools.com/howto/howto_website_static.asp) 要素を持ついくつかの Webサイトをスクレイピングしてみましょう。まずは、[https://books.toscrape.com](https://books.toscrape.com/) から書籍タイトルと価格をスクレイピングします。

最初に、必要なデータを含む HTML 要素を特定するところから始めます。  

* 書籍タイトルの selector は `#default > div.container-fluid.page > div > div > div > section > div:nth-child(2) > ol > li:nth-child(1) > article > h3 > a` です。
* 書籍価格の selector は \*`#default > div.container-fluid.page > div > div > div > section > div:nth-child(2) > ol > li:nth-child(1) > article > div.product_price > p.price_color` です。

次に、スクレイピングコードを生成するために、具体的で詳細な ChatGPT プロンプトを作成します。そのために、必要な Python パッケージをインストールするよう ChatGPT に指示し、（先ほど特定した）HTML selector の抽出方法を案内し、データを Excel ファイルとして保存するよう依頼します。 

> **Tip**:
> 
> ChatGPT のバージョンを GPT-4o に設定するのを忘れないでください。

ChatGPT には、例えば次のように入力できます。

> You are a web scraping expert utilizing Python’s Beautiful Soup library and any necessary automation tools. I want you to generate a detailed step-by-step script to scrape `https://books.toscrape.com`; kindly do the following:
> 
> 1.  Install any necessary Python packages.
> 2.  Extract the book title, which has the HTML selector `#default > div.container-fluid.page > div > div > div > section > div:nth-child(2) > ol > li:nth-child(1) > article > h3 > a`, and the book price (only the number, nothing else), which has the selector `#default > div.container-fluid.page > div > div > div > section > div:nth-child(2) > ol > li:nth-child(1) > article > div.product_price > p.price_color`.
> 3.  Extract the list of all book titles and prices and store it in an Excel file.
> 4.  Scrape the first page only.

なお、返答は多少異なる場合があります。

レスポンスの中で ChatGPT は、Webスクレイピングを実行する Beautiful Soup ライブラリと、データを Excel として保存できる包括的なデータ分析ライブラリである [pandas library](https://pandas.pydata.org/) をインストールするために、次のコマンドを実行するよう指示します。

```bash
pip3 install requests beautifulsoup4 pandas openpyxl
```

ChatGPT は [openpyxl](https://pypi.org/project/openpyxl/) のインストールも提案しますが、ここでは不要です。

また、ChatGPT は次の Webスクレイピングコードも提供します。

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

# URL of the website
url = 'https://books.toscrape.com'

# Send a GET request to the website
response = requests.get(url)
if response.status_code == 200:
    # Parse the HTML content of the page
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # Initialize lists to store the titles and prices
    book_titles = []
    book_prices = []

    # Extract the first book title and price using the provided selectors
    book_title_selector = "#default > div.container-fluid.page > div > div > div > section > div:nth-child(2) > ol > li:nth-child(1) > article > h3 > a"
    book_price_selector = "#default > div.container-fluid.page > div > div > div > section > div:nth-child(2) > ol > li:nth-child(1) > article > div.product_price > p.price_color"

    title_element = soup.select_one(book_title_selector)
    price_element = soup.select_one(book_price_selector)
    
    if title_element and price_element:
        book_titles.append(title_element.get('title'))
        book_prices.append(price_element.text.strip()[1:])

    # Extract all book titles and prices on the first page
    books = soup.select('section > div:nth-child(2) > ol > li')
    for book in books:
        title = book.select_one('h3 > a')
        price = book.select_one('div.product_price > p.price_color')
        if title and price:
            book_titles.append(title.get('title'))
            book_prices.append(price.text.strip()[1:])

    # Create a DataFrame from the lists
    df = pd.DataFrame({
        'Title': book_titles,
        'Price': book_prices
    })

    # Save the DataFrame to an Excel file
    df.to_excel('books_scraped.xlsx', index=False)

    print('Data has been successfully scraped and saved to books_scraped.xlsx')
else:
    print('Failed to retrieve the webpage')
```

このコードは、Webサイトデータをスクレイピングするための HTTP 呼び出しを扱う `requests` モジュールをインポートします。また、スクレイピングしたデータを Excel ファイルに保存するために pandas パッケージもインポートします。コードでは Webサイトのベース URL を定義し、必要なデータを抽出するために使用する HTML selector を指定しています。  

次に、コードは Webサイトのコンテンツを取得し、`soup` という名前の `BeautifulSoup` オブジェクトとして保存します。特定した HTML selector を使用して、`soup` オブジェクトから書籍タイトルと価格をリストとして抽出します。最後に、これらのリストから pandas DataFrame を作成し、データを Excel ファイルとして保存します。  

次に、コードを `books_scraping.py` というファイル名で保存し、コマンドラインから `python3 books_scraping.py` を実行する必要があります。このコードにより、`books_scraping.py` と同じディレクトリに `books_scraped.xlsx` という Excel ファイルが生成されます。

![The generated Excel file named books_scraped.xlsx](https://github.com/bright-jp/web-scraping-with-chatgpt/blob/main/images/The-generated-Excel-file-named-books_scraped.xlsx-1024x684.png)

次に、引用文のコレクションを含む別のシンプルな Webサイト https://quotes.toscrape.com をスクレイピングしてみましょう。

今回も、まずは手順の流れを特定するところから始めます。

Chrome を使用している場合は、次の HTML selector を見つけてください。

* **quote** をスクレイピングするための `body > div.container > div:nth-child(2) > div.col-md-8 > div:nth-child(1) > span.text`
* **author** を特定するための `body > div.container > div:nth-child(2) > div.col-md-8 > div:nth-child(1) > span:nth-child(2) > small`

スクレイピングしたい HTML selector を特定したら、スクレイピングコードを生成するための詳細なプロンプトを作成します。

> You are a web scraping expert utilizing Python’s Beautiful Soup library and any necessary automation tools. I want you to generate a detailed step-by-step script to scrape `https://quotes.toscrape.com`. Kindly do the following:
> 
> 1.  Install any necessary Python packages.
> 2.  Extract the quote, which has the HTML selector `body > div.container > div:nth-child(2) > div.col-md-8 > div:nth-child(1) > span.text`, and the author name, which has the selector `body > div.container > div:nth-child(2) > div.col-md-8 > div:nth-child(1) > span:nth-child(2) > small`.
> 3.  Extract the list of all quotes and authors and store it in an Excel file.
> 4.  Scrape the first page only.

この情報を入力すると、ChatGPT は Beautiful Soup、pandas、openpyxl ライブラリをインストールする次のコマンドを提示するはずです。

```bash
pip3 install requests beautifulsoup4 pandas openpyxl
```

ChatGPT は次の Webスクレイピングコードも提示するはずです。

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd

# Step 1: Fetch the web page content
url = "https://quotes.toscrape.com"
response = requests.get(url)
html_content = response.text

# Step 2: Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(html_content, 'html.parser')

# Step 3: Extract the specific quote and author
quote_selector = "body > div.container > div:nth-child(2) > div.col-md-8 > div:nth-child(1) > span.text"
author_selector = "body > div.container > div:nth-child(2) > div.col-md-8 > div:nth-child(1) > span:nth-child(2) > small"

specific_quote = soup.select_one(quote_selector).get_text()
specific_author = soup.select_one(author_selector).get_text()

print(f"Specific Quote: {specific_quote}")
print(f"Specific Author: {specific_author}")

# Step 4: Extract all quotes and authors
quotes = soup.find_all('div', class_='quote')

all_quotes = []
for quote in quotes:
  text = quote.find('span', class_='text').get_text()
  author = quote.find('small', class_='author').get_text()
  all_quotes.append({"quote": text, "author": author})

# Step 5: Save the quotes to an Excel file
df = pd.DataFrame(all_quotes)
df.to_excel('quotes.xlsx', index=False)

print("Quotes have been saved to quotes.xlsx")
soup = BeautifulSoup(html_content, 'html.parser')

# Step 3: Extract the specific quote and author
quote_selector = "body > div.container > div:nth-child(2) > div.col-md-8 > div:nth-child(1) > span.text"
author_selector = "body > div.container > div:nth-child(2) > div.col-md-8 > div:nth-child(1) > span:nth-child(2) > small"

specific_quote = soup.select_one(quote_selector).get_text()
specific_author = soup.select_one(author_selector).get_text()

print(f"Specific Quote: {specific_quote}")
print(f"Specific Author: {specific_author}")

# Step 4: Extract all quotes and authors
quotes = soup.find_all('div', class_='quote')

all_quotes = []
for quote in quotes:
    text = quote.find('span', class_='text').get_text()
    author = quote.find('small', class_='author').get_text()
    all_quotes.append({"quote": text, "author": author})

# Step 5: Save the quotes to an Excel file
df = pd.DataFrame(all_quotes)
df.to_excel('quotes.xlsx', index=False)

print("Quotes have been saved to quotes.xlsx")
```

このコードを `quotes_scraping.py` というファイル名で保存し、コマンドラインから `python3 books_scraping.py` を実行してください。このコードにより、`quotes_scraping.py` と同じディレクトリに `quotes_scraped.xlsx` という Excel ファイルが生成されます。生成された Excel ファイルを開くと、次のようになっているはずです。

![Generated Excel file with quotes and authors](https://github.com/bright-jp/web-scraping-with-chatgpt/blob/main/images/Generated-Excel-file-with-quotes-and-authors-1024x226.png)

## Scraping Complex Websites

複雑な Webサイトのスクレイピングは、動的コンテンツが JavaScript によって読み込まれることが多く、[`requests`](https://docs.python-requests.org/en/latest/) や `BeautifulSoup` のようなツールでは対応できないため、難しくなる場合があります。これらのサイトでは、すべてのデータにアクセスするために、ボタンのクリックやスクロールといった操作が必要になることがあります。  

この課題を克服するために、[WebDriver](https://www.selenium.dev/documentation/webdriver/) を使用できます。WebDriver はブラウザのようにページをレンダリングし、ユーザー操作をシミュレートすることで、一般的なユーザーと同様にすべてのコンテンツへアクセスできるようにします。  

例えば、[Yelp](https://www.yelp.com/) は、動的なページ生成に依存し、複数のユーザー操作を必要とする、事業者向けのクラウドソース型レビュー Webサイトです。このケースでは、ChatGPT を使って、ストックホルムの事業者一覧とその評価を取得するスクレイピングスクリプトを生成します。  

Yelp をスクレイピングするワークフローは次のとおりです。

1. スクリプトで使用するロケーション入力テキストボックスの selector を見つけます。この例では `#search_location` です。ロケーション検索ボックスに「Stockholm」と入力し、その後検索ボタンの selector を見つけます。この例では `#header_find_form > div.y-css-1iy1dwt > button` です。検索ボタンをクリックして検索結果を表示します。これには数秒かかる場合があります。事業者名を含む selector（_ie_ `#main-content > ul > li:nth-child(3) > div.container_\_09f24_\_FeTO6.hoverable_\_09f24_\__UXLO.y-css-way87j > div > div.y-css-cxcdjj > div:nth-child(1) > div.y-css-1iy1dwt > div:nth-child(1) > div > div > h3 > a`）を見つけます。  

![Getting the selector for the business name](https://github.com/bright-jp/web-scraping-with-chatgpt/blob/main/images/Getting-the-selector-for-the-business-name.png)

2. 事業者の評価を含む selector（_ie_ `#main-content > ul > li:nth-child(3) > div.container_\_09f24_\_FeTO6.hoverable_\_09f24_\__UXLO.y-css-way87j > div > div.y-css-cxcdjj > div:nth-child(1) > div.y-css-1iy1dwt > div:nth-child(2) > div > div > div > div.y-css-ohs7lg > span.y-css-jf9frv`）を見つけます。  

![Getting the selector for the business average review](https://github.com/bright-jp/web-scraping-with-chatgpt/blob/main/images/Getting-the-selector-for-the-business-average-review.png)

3. **Open Now** ボタンの selector を見つけます。ここでは `#main-content > div.stickyFilterOnSmallScreen_\_09f24_\_UWWJ3.hideFilterOnLargeScreen_\_09f24_\_ilqIP.y-css-9ze9ku > div > div > div > div > div > span > button:nth-child(3) > span` です。  
    
![Open Now button selector](https://github.com/bright-jp/web-scraping-with-chatgpt/blob/main/images/Open-Now-button-selector.png)
    
4. 後でアップロードできるように Webページのコピーを保存します。これは、プロンプトの文脈を ChatGPT が理解するのに役立ちます。Chrome では、右上の三点メニューをクリックし、**Save** と **Share > Save Page As** をクリックすると実行できます。  
    
![Save web page in Chrome](https://github.com/bright-jp/web-scraping-with-chatgpt/blob/main/images/Save-web-page-in-Chrome.png)    

次に、先ほど抽出した selector の値を使って、ChatGPT がスクレイピングスクリプトを生成する際の指針となる詳細なプロンプトを作成する必要があります。

> You are a web scraping expert. I want you to scrape https://www.yelp.com/ to extract specific information. Follow these steps before scraping:
> 
> 1.  Clear the box with the selector `#search_location`.
> 2.  Type “Stockholm” in the search box with the selector `#search_location`.
> 3.  Click the button with the selector `#header_find_form > div.y-css-1iy1dwt > button`.
> 4.  Account for a few seconds for the page to load the search results.
> 5.  Click the button **Open Now**, which has the selector `#main-content > div.stickyFilterOnSmallScreen_\_09f24_\_UWWJ3.hideFilterOnLargeScreen_\_09f24_\_ilqIP.y-css-9ze9ku > div > div > div > div > div > span > button:nth-child(3) > span`, to filter the businesses that are open now.
> 6.  Create a list with business names; the business name has the selector `#main-content > ul > li:nth-child(3) > div.container_\_09f24_\_FeTO6.hoverable_\_09f24_\__UXLO.y-css-way87j > div > div.y-css-cxcdjj > div:nth-child(1) > div.y-css-1iy1dwt > div:nth-child(1) > div > div > h3 > a`.
> 7.  Create a list with the business reviews; the reviews have the selector `#main-content > ul > li:nth-child(3) > div.container_\_09f24_\_FeTO6.hoverable_\_09f24_\__UXLO.y-css-way87j > div > div.y-css-cxcdjj > div:nth-child(1) > div.y-css-1iy1dwt > div:nth-child(2) > div > div > div > div.y-css-ohs7lg > span.y-css-jf9frv`.
> 8.  Create an Excel file with the business names and their reviews.
> 9.  Make sure to install all the relevant Python packages.
> 10.  Use WebDriver from Selenium to automate the process.
> 11.  Refer to the attached copy of the results HTML page to give you context about the HTML format.

この情報を入力すると、ChatGPT は WebDriver をサポートする Web 自動化フレームワークである Selenium と、pandas をインストールするよう指示します。

```bash
pip3 install selenium pandas
```

また、ChatGPT はいくつかのスクレイピングコードと説明も提供します。

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import pandas as pd
import time

# Initialize the WebDriver
driver = webdriver.Chrome()

try:
    # Open Yelp
    driver.get('https://www.yelp.com/')

    # Clear the location search box
    location_box = driver.find_element(By.CSS_SELECTOR, '#search_location')
    location_box.clear()

    # Enter "Stockholm" in the location search box
    location_box.send_keys('Stockholm')

    # Click the search button
    search_button = driver.find_element(By.CSS_SELECTOR, '#header_find_form > div.y-css-1iy1dwt > button')
    search_button.click()

    # Wait for search results to load
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.CSS_SELECTOR, '#main-content'))
    )

    # Click on "Open Now" filter
    open_now_button = driver.find_element(By.CSS_SELECTOR, '#main-content > div.stickyFilterOnSmallScreen__09f24__UWWJ3.hideFilterOnLargeScreen__09f24__ilqIP.y-css-9ze9ku > div > div > div > div > div > span > button:nth-child(3) > span')
    open_now_button.click()

    # Wait for the filter results to load
    time.sleep(5)

    # Extract business names and reviews
    business_names = []
    business_reviews = []

    businesses = driver.find_elements(By.CSS_SELECTOR, '#main-content > ul > li')

    for business in businesses:
        try:
            name = business.find_element(By.CSS_SELECTOR, 'div.container__09f24__FeTO6.hoverable__09f24___UXLO.y-css-way87j > div > div.y-css-cxcdjj > div:nth-child(1) > div.y-css-1iy1dwt > div:nth-child(1) > div > div > h3 > a').text
            review = business.find_element(By.CSS_SELECTOR, 'div.container__09f24__FeTO6.hoverable__09f24___UXLO.y-css-way87j > div > div.y-css-cxcdjj > div:nth-child(1) > div.y-css-1iy1dwt > div:nth-child(2) > div > div > div > div.y-css-ohs7lg > span.y-css-jf9frv').text
            business_names.append(name)
            business_reviews.append(review)
        except Exception as e:
            print(f"Error extracting data for a business: {e}")

    # Create a DataFrame and save to Excel
    df = pd.DataFrame({'Business Name': business_names, 'Reviews': business_reviews})
    df.to_excel('stockholm_businesses.xlsx', index=False)
    print("Data saved to stockholm_businesses.xlsx")

finally:
    # Close the WebDriver
    driver.quit()

```

このスクリプトを保存し、VS Code のような IDE で Python を使って実行します。コードが Chrome を起動し、Yelp に移動し、ロケーションのテキストボックスをクリアし、「Stockholm」を入力して検索ボタンをクリックし、営業中の事業者をフィルタリングしてからページを閉じることが分かるはずです。その後、スクレイピング結果は Excel ファイル `stockholm_bussinsess.xlsx` に保存されます。

![Yelp business reviews in Excel](https://github.com/bright-jp/web-scraping-with-chatgpt/blob/main/images/Yelp-business-reviews-in-Excel.png)

このチュートリアルのソースコードはすべて [GitHub](https://github.com/smarter-code/article-chatgpt-webscraping/tree/main) で利用できます。

## Conclusion

このガイドでは Yelp のスクレイピングは簡単でしたが、他の多くのシナリオでは、複雑な HTML 構造の Webスクレイピングが難しくなる場合があります。

Bright Data は、IP BAN の回避に役立つ高度なプロキシサービス、[CAPTCHA を回避して解決する Web Unlocker](https://brightdata.jp//products/web-unlocker)、自動データ抽出のための [Web Scraping APIs](https://brightdata.jp/products/web-scraper)、効率的なデータ抽出のための [Scraping Browser](https://brightdata.jp/products/scraping-browser) など、幅広いデータ収集サービスを提供しています。

今すぐ無料トライアルから始めましょう！