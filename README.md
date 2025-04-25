# 1
#SSE kullanarak manuel olarak bir json formatındaki dosyayı playwright mcp sunucusuna iletmeyi deniyorum.

#main.py

import requests
import uuid
import threading
import sseclient
import time

# Oturum kimliği oluştur
session_id = str(uuid.uuid4())
sse_url = f"http://localhost:5050/sse?sessionId={session_id}"
post_url = "http://localhost:5050/task"


# SSE'yi dinleyen thread fonksiyonu
def listen_to_sse():
    print(f"🔁 SSE dinleme başlatıldı: {sse_url}")
    try:
        response = requests.get(sse_url, stream=True)
        response.raise_for_status()  # Bağlantı hatası olursa yükselt
        client = sseclient.SSEClient(response)  # ✅ KESİN DÜZELTME
        for event in client.events():
            print(f"📩 SSE Yanıtı: {event.data}")
            if '"status":"done"' in event.data:
                break
    except requests.exceptions.RequestException as e:
        print(f"❌ SSE Bağlantı Hatası: {e}")

# SSE dinleyicisini başlat
thread = threading.Thread(target=listen_to_sse)
thread.start()

# Görev JSON'u
task = {
    "action": "open_url",
    "url": "https://www.booking.com",
    "headless": True,
    "sessionId": session_id
}

time.sleep(1)  # Gerekirse bu süreyi artırabilirsiniz

# Görevi POST ile gönder
try:
    response = requests.post(post_url, json=task)
    response.raise_for_status()
except requests.exceptions.RequestException as e:
    print(f"❌ POST Hata: {e}")
else:
    print(f"✅ POST Başarılı: {response.status_code}")
    print(f"🧠 Kullanılan sessionId: {session_id}")
    print(f"📨 Yanıt içeriği: {response.text}")

# Thread'in bitmesini bekle (isteğe bağlı)
# thread.join()


#ERROR

(venv) mac@MacBook-Pro MCP-REQUESTS % /Users/mac/Desktop/MCP-REQUESTS/venv/bin/python /Users/mac/Desktop/MCP-REQ
UESTS/main.py
🔁 SSE dinleme başlatıldı: http://localhost:5050/sse?sessionId=ef4485fc-05ef-4b2d-bbef-5dac1486a628
❌ SSE Bağlantı Hatası: Failed to parse: <Response [200]>
❌ POST Hata: 400 Client Error: Bad Request for url: http://localhost:5050/task
