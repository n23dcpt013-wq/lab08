Lab 08 – Kiểm thử ATM (Unit Test & Integration Test)
 Unit Test
File: test_withdraw.py
import mysql.connector, hashlib

def verify_pin(card_no, pin):
    conn = mysql.connector.connect(user="root", password="123456", database="atm_demo")
    cur = conn.cursor()
    cur.execute("SELECT pin_hash FROM cards WHERE card_no=%s", (card_no,))
    row = cur.fetchone()
    conn.close()
    return row and row[0] == hashlib.sha256(pin.encode()).hexdigest()

def withdraw(card_no, amount):
    conn = mysql.connector.connect(user="root", password="123456", database="atm_demo")
    cur = conn.cursor()
    try:
        conn.start_transaction()
        cur.execute("SELECT account_id, balance FROM accounts JOIN cards USING(account_id) "
                    "WHERE card_no=%s FOR UPDATE",(card_no,))
        account_id,balance = cur.fetchone()
        if balance < amount:
            raise Exception("Insufficient funds")
        cur.execute("UPDATE accounts SET balance=balance-%s WHERE account_id=%s",(amount,account_id))
        conn.commit()
        return True
    except Exception as e:
        conn.rollback()
        return False
    finally:
        conn.close()

Kết quả: 4 test case chạy thành công (PASS). Sinh ra file unit_report.html.
3. Integration Test với Selenium
File: selenium_test_login.py
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

LOGIN_URL = "https://n23dcpt013-wq.github.io/lab04/login.html"

def _open():
    d = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
    d.get(LOGIN_URL); d.maximize_window()
    time.sleep(0.4)
    return d

def _find_username(d):
    return d.find_element(By.CSS_SELECTOR, "input[type='text'], input:not([type])")

def _find_password(d):
    return d.find_element(By.CSS_SELECTOR, "input[type='password'])

def _report_validity(d):
    u = _find_username(d)
    return d.execute_script("""
        var el = arguments[0];
        var f = el.form || el.closest('form');
        if (!f) return false;
        return f.reportValidity();
    """, u)

def _validation_msgs(d):
    u = _find_username(d); p = _find_password(d)
    um = (d.execute_script("return arguments[0].validationMessage || ''", u) or "").lower()
    pm = (d.execute_script("return arguments[0].validationMessage || ''", p) or "").lower()
    return um, pm

# 3 test case: success, wrong password, empty input

Kết quả: 3 test case chạy thành công (PASS). Sinh ra file selenium_report.html.
4. Ảnh minh họa kết quả
[4passsed](https://github.com/n23dcpt013-wq/lab08/blob/main/b367c7b9-8818-4e97-bf34-112c470b4a00.png)


[3passed](https://github.com/n23dcpt013-wq/lab08/blob/main/c89e333a-a8c1-43f2-951a-2121c98f6fee.png)


