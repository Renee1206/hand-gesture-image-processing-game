# Hand Gesture Image Processing Game

一款結合 **手部姿態辨識（Hand Tracking）** 與 **影像處理（Image Processing）** 的即時互動式小遊戲。

本專案透過 Webcam 即時偵測玩家手部位置，玩家需要移動手掌觸碰畫面中的隨機目標以獲得分數。隨著分數提升，套用於圖片上的馬賽克、模糊、椒鹽雜訊與旋渦扭曲等影像處理效果會逐漸減弱，最終還原出原始圖片。

---

## 專案功能

- Webcam 即時影像擷取
- 即時手部偵測與關鍵點追蹤
- 手部與攝影機距離估測
- 隨機目標生成與碰撞判定
- 60 秒遊戲倒數計時
- 即時計分系統
- 握拳手勢暫停遊戲
- 根據遊戲進度逐步還原圖片
- 多種影像處理效果
  - Mosaic 馬賽克
  - Gaussian Blur 高斯模糊
  - Salt-and-Pepper Noise 椒鹽雜訊
  - Swirl Distortion 旋渦扭曲
- 遊戲重新開始與離開功能
- 完成遊戲後的愛心動畫效果

---

## 遊戲方式

遊戲開始後，程式會透過 Webcam 即時偵測玩家的手部。

畫面中會隨機產生一個圓形目標，玩家需要將手移動至目標範圍內，並保持在適當的攝影機距離。

成功碰觸目標後：

1. 分數增加。
2. 重新產生新的隨機目標。
3. 圖片上的影像處理效果逐步降低。
4. 原始圖片會隨著分數逐漸顯示。

玩家需要在倒數時間結束前盡可能完成圖片還原。

當圖片成功完成還原後，畫面會顯示完成效果以及愛心動畫。

---

## 手部辨識

本專案透過 `cvzone` 提供的 `HandDetector` 進行手部偵測：

```python
from cvzone.HandTrackingModule import HandDetector
```

程式設定一次偵測一隻手：

```python
detector = HandDetector(
    detectionCon=0.8,
    maxHands=1
)
```

系統會取得手部關鍵點以及 Bounding Box，並利用手部位置判斷玩家是否碰觸到畫面中的目標。

---

## 距離估測

除了判斷手部位置之外，本專案也利用手部關鍵點之間的像素距離估算手掌與攝影機之間的實際距離。

首先根據事先量測的距離資料建立二次多項式：

```python
coff = np.polyfit(x, y, 2)
```

接著利用偵測到的手部像素距離計算：

```python
distanceCM = A * distance ** 2 + B * distance + C
```

藉此判斷玩家的手是否位於適當的操作距離內。

> 距離估測結果會受到 Webcam、解析度及拍攝環境影響，因此不同設備可能需要重新進行校正。

---

## 影像處理

遊戲開始時會在原始圖片 `f.jpg` 上套用多種影像處理效果。

隨著玩家分數增加，各項效果會逐漸降低，使圖片慢慢恢復成原始狀態。

### Mosaic 馬賽克

將影像切割成多個區塊，使用區域像素資訊產生馬賽克效果。

```python
apply_mosaic(image, block_size)
```

隨著遊戲進度增加，馬賽克區塊大小會逐漸減少。

### Salt-and-Pepper Noise 椒鹽雜訊

隨機在影像中加入黑色與白色像素點，產生椒鹽雜訊效果。

```python
apply_salt_and_pepper(image, amount)
```

隨著玩家得分增加，雜訊比例會逐漸下降。

### Gaussian Blur 高斯模糊

使用 Gaussian Filter 對圖片進行模糊處理。

```python
apply_fuzzy(image)
```

遊戲進行過程中會逐漸降低模糊程度。

### Swirl Distortion 旋渦扭曲

利用座標轉換方式改變影像像素位置，使圖片產生旋渦扭曲效果。

```python
rotate()
rotate1()
```

接近遊戲完成時，旋渦效果會逐漸減弱並恢復原始影像。

---

## 遊戲操作

| 操作 | 說明 |
|---|---|
| 移動手掌 | 控制手部位置並碰觸目標 |
| 握拳 | 暫停遊戲 |
| 張開手掌 | 繼續遊戲 |
| `R` | 重新開始遊戲 |
| `Q` | 離開遊戲 |

---

## 遊戲流程

```text
開始遊戲
   │
   ▼
開啟 Webcam
   │
   ▼
偵測手部
   │
   ├── 偵測到握拳 ──► 暫停遊戲
   │
   ▼
估算手部距離
   │
   ▼
判斷是否碰觸目標
   │
   ├── 成功碰觸
   │      │
   │      ├── 分數增加
   │      ├── 產生新目標
   │      └── 降低圖片失真效果
   │
   ▼
判斷時間與遊戲進度
   │
   ├── 時間結束 ──► Game Over
   │
   └── 圖片完成還原
             │
             ▼
       顯示完成畫面
       與愛心動畫
```

---

## 使用技術

- **Python**
- **OpenCV**
  - Webcam 影像擷取
  - 影像處理
  - 畫面繪製
- **cvzone**
  - Hand Tracking
  - 手部 Bounding Box 與 Landmark 處理
- **MediaPipe**
  - 手部關鍵點辨識
- **NumPy**
  - 數值運算
  - 多項式距離估測
  - 影像矩陣處理

---

## 專案結構

```text
hand-gesture-image-processing-game/
│
├── final_project.py
├── f.jpg
└── hand_landmarker.task
```

### `final_project.py`

專案主要程式，包含：

- Webcam 影像擷取
- 手部偵測
- 距離估測
- 遊戲邏輯
- 計分系統
- 倒數計時
- 影像處理
- 手勢暫停功能
- 遊戲完成動畫

### `f.jpg`

遊戲中使用的原始圖片。

程式會讀取此圖片並加入不同影像處理效果，玩家透過完成目標逐步將圖片還原。

### `hand_landmarker.task`

MediaPipe Hand Landmarker 模型檔案。

> 目前 `final_project.py` 主要透過 `cvzone.HandTrackingModule.HandDetector` 進行手部偵測，程式中並未直接載入此 `.task` 模型檔案。

---

## 安裝方式

### 1. Clone 專案

```bash
git clone https://github.com/Renee1206/hand-gesture-image-processing-game.git
cd hand-gesture-image-processing-game
```

### 2. 安裝套件

```bash
pip install opencv-python cvzone mediapipe numpy
```

### 3. 執行程式

```bash
python final_project.py
```

執行前請確認電腦具有可正常使用的 Webcam。

---

## Requirements

- Python 3
- Webcam
- OpenCV
- cvzone
- MediaPipe
- NumPy

---

## 距離校正

目前程式使用預先建立的校正資料進行距離估測：

```python
x = [300, 245, 200, 170, 145, 130, 112, 103, 93, 87, 80, 75, 70, 67, 62, 59, 57]
y = [20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95, 100]
```

其中：

- `x`：手部在畫面中的像素距離
- `y`：實際量測距離（cm）

不同攝影機的視角與焦距可能不同，如距離判斷誤差較大，可以重新建立校正資料。

---
