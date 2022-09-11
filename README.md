# web_cam

```
 pip install opencv-contrib-python
``` 

views.py

```
import cv2
from django.http import StreamingHttpResponse
from django.shortcuts import render
from django.views import View
import time


# ストリーミング画像・映像を表示するview
class IndexView(View):
    def get(self, request):
        return render(request, 'index.html', {})

# ストリーミング画像を定期的に返却するview
def video_feed_view():
    return lambda _: StreamingHttpResponse(generate_frame(), content_type='multipart/x-mixed-replace; boundary=frame')

# フレーム生成・返却する処理
def generate_frame():
    # カスケード分類器読み込み ## https://opencv.org/releases/
    HAAR_FILE = "/Downloads/opencv-4.6.0/data/haarcascades/haarcascade_frontalface_default.xml"
    cascade = cv2.CascadeClassifier(HAAR_FILE)


    capture = cv2.VideoCapture(0)

    while True:
        if not capture.isOpened():
            print("Capture is not opened.")
            break
        # カメラからフレーム画像を取得(retはboolean, frameは顔画像データ)
        ret, frame = capture.read()
        if not ret:
            print("Failed to read frame.")
            break
        # グレースケール変換
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        face = cascade.detectMultiScale(gray)

        # 顔領域を赤色の矩形で囲む
        for x, y, w, h in face:
            cv2.rectangle(frame, (x, y), (x + w, y + h), (0, 0, 255), 1)
        ret, jpeg = cv2.imencode('.jpg', frame)

        # フレーム画像バイナリに変換
        byte_frame = jpeg.tobytes()
        # フレーム画像のバイナリデータをユーザーに送付する
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + byte_frame + b'\r\n\r\n')
        time.sleep(1)
        if ret == True:
            cv2.imwrite('save.jpg', frame)
            # ここでkpas apiに送って認証
            # 認証が完了したら
            print("save")
    capture.release()
```

#camera.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Live streaming</title>
</head>
<body>
    <div>
    <div id="image_area">
        <img src="/video_feed" width="810" height="540"/>
    </div>

</div>
</body>
</html>
```

# urls
```
urlpatterns = [
    path('', views.video_feed_view(), name='video_feed'),
]
```
