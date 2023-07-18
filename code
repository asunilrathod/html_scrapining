import requests
from bs4 import BeautifulSoup
import re
import pandas as pd
import sqlite3
from urllib.parse import urljoin

# Set the base URL and initialize page variables
base_url = 'https://books.toscrape.com/catalogue/category/books_1/'
page_url = base_url

# Create an empty list to store book information
book_data = []

while True:
    # Send a GET request to the current page
    response = requests.get(page_url)

    if response.status_code == 200:
        # Parse the HTML content using BeautifulSoup
        soup = BeautifulSoup(response.content, 'html.parser')

        # Extract information from the parsed HTML
        # For example, let's extract the titles, prices, ratings, and stock information of the books
        books = soup.find_all('article', class_='product_pod')

        for book in books:
            title = book.find('h3').text
            price = book.find('p', class_='price_color').text
            rating_text = book.find('p', class_='star-rating')['class'][1]

            # Convert the rating to an integer
            rating_mapping = {
                'One': 1,
                'Two': 2,
                'Three': 3,
                'Four': 4,
                'Five': 5
            }
            rating = rating_mapping.get(rating_text)

            if rating == 5:
                # Visit the book's detailed page
                book_url = book.find('a')['href']
                book_full_url = urljoin(base_url, book_url)
                book_response = requests.get(book_full_url)

                if book_response.status_code == 200:
                    book_soup = BeautifulSoup(book_response.content, 'html.parser')

                    # Extract the stock information
                    stock = book_soup.find('p', class_='instock availability').text.strip()

                    # Extract the number from the stock information
                    stock_number = re.search(r'\d+', stock).group()

                    # Determine the stock status
                    if int(stock_number) >= 5:
                        stock_status = "Good number of stocks"
                    else:
                        stock_status = "Limited stocks"

                    # Append the book information to the list
                    book_data.append({
                        'Title': title,
                        'Price': price,
                        'Rating': rating,
                        'Stock': stock_number,
                        'Stock Status': stock_status
                    })

        # Check if there is a next page
        next_page = soup.find('li', class_='next')
        if next_page is None:
            break

        next_page_link = next_page.find('a')['href']
        page_url = urljoin(base_url, next_page_link)

    else:
        print(f'Failed to retrieve page: {page_url}')
        break

# Create a pandas DataFrame from the book data
df = pd.DataFrame(book_data)

# Establish a connection to the database
conn = sqlite3.connect('books.db')

# Write the DataFrame to the database
df.to_sql('books', conn, if_exists='replace', index=False)

# Close the database connection
conn.close()
