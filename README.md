# Prog2_11th
プログラミングⅡ　第11週　Gitの体験のために

この資料は，Git(GitHub)を体験してもらうための資料です．
以下の手順に従って実行してください

１．このページをForkする（つまり，保存してあるGoogle Colabのファイル（.ipynb)を自分のGitHubにダウンロードする．この時点ではGitHub上にデータがあります）

２．ファイルを自分のパソコンにダウンロードし，ファイルの指示に従ってプログラムする

３．プログラム結果を保存し，ファイルをGitHubに保存する

４．GitHubに掲載したプログラムを私宛ににpull requestする

各操作が分からない人は解説動画を参照してください．


import cv2
import time
import RPi.GPIO as GPIO
import sys
import csv
# ポート番号の定義
Led_pin=25


GPIO.setmode(GPIO.BCM)
GPIO.setup(26,GPIO.IN,pull_up_down=GPIO.PUD_DOWN)
GPIO.setup(21,GPIO.OUT)
pwm=GPIO.PWM(21,200)
GPIO.setup(Led_pin,GPIO.OUT)
data=[0,0,0,0,0,0,0]
jweek=["月曜日","火曜日","水曜日","木曜日","金曜日","土曜日","日曜日"]
header=jweek
with open("sample.csv","w")as f:
    writer=csv.writer(f)
    writer.writerow(header)
preweek=0
renew=0
# 画像に動きがあったか調べる関数
def check_image(img1, img2, img3):
    # グレイスケール画像に変換 --- (*6)
    gray1 = cv2.cvtColor(img1, cv2.COLOR_RGB2GRAY)
    gray2 = cv2.cvtColor(img2, cv2.COLOR_RGB2GRAY)
    gray3 = cv2.cvtColor(img3, cv2.COLOR_RGB2GRAY)
    # 絶対差分を調べる --- (*7)
    diff1 = cv2.absdiff(gray1, gray2)
    diff2 = cv2.absdiff(gray2, gray3)
    # 論理積を調べる --- (*8)
    diff_and = cv2.bitwise_and(diff1, diff2)
    # 白黒二値化 --- (*9)
    _, diff_wb = cv2.threshold(diff_and, 30, 255, cv2.THRESH_BINARY)
    # ノイズの除去 --- (*10)
    diff = cv2.medianBlur(diff_wb, 5)
    return diff

# カメラから画像を取得する
def get_image(cam):
    img = cam.read()[1]
    img = cv2.resize(img, (600, 400))
    return img

def wait_until(v): #スイッチの入力検知
  while GPIO.input(26) != v:
    time.sleep(0.01)
def LED_on():
    GPIO.output(Led_pin,GPIO.HIGH)
def LED_off():
    GPIO.output(Led_pin,GPIO.LOW)
    
while True:
    # カメラのキャプチャを開始
    cam = cv2.VideoCapture(0)
    # フレームの初期化 --- (*1)
    img1 = img2 = img3 = get_image(cam)
    th = 100
    react=0
    print("1:月曜, 2:火曜, 3:水曜, 4:木曜, 5:金曜, 6:土曜, 7:日曜")
    while True:
        week=int(input("勉強する曜日の数字を入力してね"))
        if week<=7 & week>=1:break
        else: continue
    if week<=preweek:
        print(f"{jweek[week-1]}にデータを追加しますか？")
        while True:
            answer=int(input("1:はい, 2:いいえ"))
            if answer==1 or answer==2:break
        if answer==2:
            print("先週はお疲れ様でした。先週のデータは以下です")
            with open("sample.csv","a")as f:
                writer=csv.writer(f)
                writer.writerow(data)
            data=[0,0,0,0,0,0,0]
            with open("sample.csv","r")as f:
                print(f.read())
        ###データ表示
            renew=0
        else:
            print("かしこまりました、それでは頑張ってください")
            renew=1
    time_study=int(input("１セット、何分勉強しますか？"))
    time_rest=int(input("１セット、何分休憩しますか？"))
    studytime=time_study*60
    print("準備が良ければ、スイッチを押してください")
    wait_until(GPIO.HIGH)
    print("計測を開始します！頑張ってください！")
    time_first=time.time()
    notime=0
    while True:
    #差分を調べる
        diff = check_image(img1, img2, img3)
    #差分がthの値以上なら動きがあったと判定
        cnt = cv2.countNonZero(diff)
        if cnt > th:
            #cv2.imshow('PUSH ENTER KEY', img3)
            react=0
        else:
            #cv2.imshow('PUSH ENTER KEY', diff)
            react+=1
        # 比較用の画像を保存
        img1, img2, img3 = (img2, img3, get_image(cam))
        print(react)
        ### お手洗いなどの中断機能
        if GPIO.input(26)==True:
            notime_first=time.time()
            print("計測を中断します")
            time.sleep(2)
            wait_until(GPIO.HIGH)
            print("計測を再開します、頑張ってください！")
            time.sleep(1)
            notime+=time.time()-notime_first
            #break
            
        ### 一定時間カメラの反応がない場合
        if react>=50:
            print("動きが検知されません！スイッチを押してください！！")
            notime_first=time.time()
            LED_on()
            pwm.ChangeFrequency(200)
            pwm.start(50)
            wait_until(GPIO.HIGH)
            pwm.stop()
            LED_off()
            time.sleep(0.5)
            react=0
            notime+=time.time()-notime_first
        ### 勉強時間が設定した時間に到達した場合
        if time.time() - time_first - notime >= 10:
            print(f"お疲れ様です！ {time_study}分勉強しました.")
            print(f"{time_rest}分休憩してください")
            data[week-1]+=time_study
            preweek=week
            break
            #time.sleep(time_rest*60)
    cam.release()
    cv2.destroyAllWindows()



