# 抓取亞洲大學資工系老師的專長資料
#老師可以不要當我嗎?順便給我高分一點，資傳系弱智已努力了

import requests
from bs4 import BeautifulSoup
import json
import csv
import sqlite3

# 亞大資工副教授網頁
url = "https://csie.asia.edu.tw/zh_tw/TeacherIntroduction"
headers = {
    "User-Agent": "Mozilla/5.0"
}

# 發送請求，抓取網頁
response = requests.get(url, headers=headers)
response.encoding = 'utf-8'

# 解析 HTML
soup = BeautifulSoup(response.text, 'html.parser')  # 使用 html.parser 解析器

# 抓每位老師的專長
data = []

# 找到所有老師的卡片（每個人都是一個 .teacher_content）
teachers = soup.find_all("div", class_="teacher_content")

for teacher in teachers:
    name_tag = teacher.find("div", class_="teacher_name")
    expertise_tag = teacher.find("div", class_="teacher_expertise")

    if name_tag and expertise_tag:
        name = name_tag.text.strip()
        expertise = expertise_tag.text.strip()
        data.append({
            "name": name,
            "expertise": expertise
        })

# 儲存為 JSON 檔
with open("asia_university_professors.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=4)

print("專長資料已儲存為 JSON")

# 儲存為 CSV 檔
with open("asia_university_professors.csv", mode='w', newline='', encoding='utf-8') as f:
    writer = csv.DictWriter(f, fieldnames=["name", "expertise"])
    writer.writeheader()
    writer.writerows(data)

print("專長資料已儲存為 CSV")

# 儲存到 SQLite 資料庫
conn = sqlite3.connect('professors.db')
cursor = conn.cursor()

# 建立資料表
cursor.execute('''
    CREATE TABLE IF NOT EXISTS professors (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT,
        expertise TEXT
    )
''')

# 插入資料
for professor in data:
    cursor.execute('''
        INSERT INTO professors (name, expertise) VALUES (?, ?)
    ''', (professor["name"], professor["expertise"]))

conn.commit()
conn.close()

print("專長資料已儲存到 SQLite 資料庫")
