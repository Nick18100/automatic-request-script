from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup
import csv
import time
import os
import re
os.environ['no_proxy'] = 'localhost,127.0.0.1'

url = "http://127.0.0.1:8000"
target_url = ""
number_of_requests = 50
output_file = "debug_toolbar_data.csv"

driver = webdriver.Chrome() 

def parse_debug_toolbar(html):
    soup = BeautifulSoup(html, 'html.parser')
    
    sql_panel = soup.find('div', {'id': 'SQLPanel', 'class': 'djdt-panelContent'})    
    if sql_panel:
        panel = sql_panel.find('div', {'class': 'djDebugPanelContent'}).find('div', {'class': 'djdt-scroll'}).find('ul').get_text()
        pattern = r'(\d+\.\d+|\d+)'
        matches = re.findall(pattern, panel)

        ms_time = float(matches[0])
        total_queries = int(matches[1])

        try:
            similar_queries = int(matches[2])
        except Exception: 
            similar_queries = 0

        try:
            duplicate_queries = int(matches[3])
        except Exception:
            duplicate_queries = 0

        print(f"ms_time: {ms_time}")
        print(f"total_queries: {total_queries}")
        print(f"similar_queries: {similar_queries}")
        print(f"duplicate_queries: {duplicate_queries}")

        count = soup.find('div', {'id': 'djDebugToolbar'}).find('li', {'id': 'djdt-TimerPanel'}).find('small').get_text()
        matches = re.findall(pattern, count)
        total_time = float(matches[0])
        print(f"total_time: {ms_time}")

        cache_data = soup.find('div', {'id': 'djDebugToolbar'}).find('li', {'id': 'djdt-CachePanel'}).find('small').get_text()
        matches = re.findall(pattern, cache_data)
        cache_calls = int(matches[0])
        cache_time = float(matches[1])

        print(f"cache_calls: {cache_calls}")
        print(f"cache_time: {cache_time}")

        return {
            'total_time': total_time,
            'query_time': ms_time,
            'total_queries': total_queries,
            'similar_queries': similar_queries,
            'duplicate_queries': duplicate_queries,
            'cache_calls': cache_calls,
            'cache_time': cache_time
        }
    return None

def scrape_page(url):
    driver = webdriver.Chrome()
    try:
        driver.get(url + '/accounts/login/')
        WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.ID, "form-submit")))

        login_div = driver.find_element(By.ID, 'login-div')
        driver.execute_script("arguments[0].style.display='block';", login_div)

        submit = driver.find_element(By.ID, 'form-submit')
        email = driver.find_element(By.ID, 'email')
        password = driver.find_element(By.ID, 'password')

        email.send_keys('email')
        password.send_keys('password')
        submit.send_keys(Keys.RETURN)

        WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CLASS_NAME, "notification-positive")))
        url = url + target_url
        driver.get(url)
        time.sleep(2)

        button = WebDriverWait(driver, 50).until(
           EC.element_to_be_clickable((By.ID, "djShowToolBarButton"))
        )
        button.click()
        sql_panel_link = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.CLASS_NAME, "SQLPanel"))
        )
        sql_panel_link.click()
        time.sleep(3)

        html = driver.page_source
        return parse_debug_toolbar(html)

    except Exception as e:
        print("Fehler: ", repr(e))
        return None

    finally:
        driver.quit()
    
def main():
    data = []
    for i in range(number_of_requests):
        print(f"request {i+1} of {number_of_requests}...")
        result = scrape_page(url)
        if result:
            data.append(result)
        time.sleep(1)

    sum_total_time = 0
    sum_query_time = 0
    sum_total_queries = 0
    sum_similar_queries = 0
    sum_duplicate_queries = 0
    sum_cache_calls = 0
    sum_cache_time = 0
    row_count = 0

    with open(output_file, mode='w', newline='') as file:
        writer = csv.DictWriter(file, fieldnames=['total_time', 'query_time', 'total_queries', 'similar_queries', 'duplicate_queries', 'cache_calls', 'cache_time'])
        writer.writeheader()
        for row in data:
            writer.writerow(row)

            sum_total_time += row['total_time']
            sum_query_time += row['query_time']
            sum_total_queries += row['total_queries']
            sum_similar_queries += row['similar_queries']
            sum_duplicate_queries += row['duplicate_queries']
            sum_cache_calls += row['cache_calls']
            sum_cache_time += row['cache_time']
            row_count += 1

    avg_total_time = sum_total_time / row_count
    avg_query_time = sum_query_time / row_count
    avg_total_queries = sum_total_queries / row_count
    avg_similar_queries = sum_similar_queries / row_count
    avg_duplicate_queries = sum_duplicate_queries / row_count
    avg_cache_calls = sum_cache_calls / row_count
    avg_cache_time = sum_cache_time / row_count


    with open(output_file, mode='a', newline='') as file:
        writer = csv.DictWriter(file, fieldnames=['total_time', 'query_time', 'total_queries', 'similar_queries', 'duplicate_queries', 'cache_calls', 'cache_time'])
        
        writer.writerow({})
        
        writer.writerow({
            'total_time': avg_total_time,
            'query_time': avg_query_time,
            'total_queries': avg_total_queries,
            'similar_queries': avg_similar_queries,
            'duplicate_queries': avg_duplicate_queries,
            'cache_calls': avg_cache_calls,
            'cache_time': avg_cache_time
        })

if __name__ == "__main__":
    main()

    driver.quit()
