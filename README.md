# 라즈베리파이에서의 키오스크 템플릿

정말 삽질 많이했다.

## 기본 컨셉

라즈베리파이의 OS (raspbian) 부팅 이후, 자동으로 chromium-browser를 열어서 특정 URL로 접속하도록 한다. 


## autostart

autostart 라는 파일은 /etc/xdg/lxsession/LXDE-pi/autostart 라는 경로에 존재한다.

라즈베리파이 기반 키오스크에서 제일 그지같은 점은 다음 두 부분이였다.

1. 크로미움 브라우저를 열었을 때. 정상적으로 부팅이 되지 않으면 에러 메시지가 화면을 가린다. 

2. 10분마다 라즈베리파이와 연결된 모니터가 꺼진다.

내가 작성한 autostart 스크립트는 다음 두 부분을 해결하였다.

```text
@lxpanel --profile LXDE-pi
@pcmanfm --desktop --profile LXDE-pi

@/bin/sleep 20
@/usr/bin/chromium-browser --disk-cache-size=2048 --noerrdialogs --kiosk --disable-pinch --overscroll-history-navigation=0 -app=http://127.0.0.1:8888

@xset -display :0 s off
@xset -display :0 -dpms
@xset -display :0 s noblank
```

여기서 

`@/usr/bin/chromium-browser --disk-cache-size=2048 --noerrdialogs --kiosk --disable-pinch --overscroll-history-navigation=0 -app=http://127.0.0.1:8888` 

위 명령어는 크로미움 브라우저를 키오스크에 아주 적절하게 사용가능하도록 여는 명령어이다.

-app 인자로 열게되면 자동으로 F11을 누른것 마냥 전체화면으로 나온다. 그리고 에러메시지도 절대 안뜬다. (개꿀)

--disk-cache-size 의 경우는. 지정 안하면 디스크에 캐시가 미친듯이 쌓여서 지정해주었다. 

`~/.cache/chromium/Default/Cache` 경로에 저장되는데. 이게 많이쌓이면 32비트 운영체제의 경우 inode 개수가 꽉차버려서 os가 맛탱이 가는 경우가 실제로 존재했다. 그래서 지정하는거다.


```
@xset -display :0 s off
@xset -display :0 -dpms
@xset -display :0 s noblank
```

위 3 라인은 화면보호기를 끄고, 에너지스타를 끄는 명령어이다. 

구글링하면 온갖 junk 정보가 난무한데. 그냥 위처럼 하면 된다. 제대로 먹었는지 확인하려면

`$ xset -display :0 q` 해보자.

```console
$ xset -display :0 q
Keyboard Control:
  auto repeat:  on    key click percent:  0    LED mask:  00000000
  XKB indicators:
    00: Caps Lock:   off    01: Num Lock:    off    02: Scroll Lock: off
    03: Compose:     off    04: Kana:        off    05: Sleep:       off
    06: Suspend:     off    07: Mute:        off    08: Misc:        off
    09: Mail:        off    10: Charging:    off    11: Shift Lock:  off
    12: Group 2:     off    13: Mouse Keys:  off
  auto repeat delay:  500    repeat rate:  33
  auto repeating keys:  00ffffffdffffbbf
                        fadfffefffedffff
                        9fffffffffffffff
                        fff7ffffffffffff
  bell percent:  50    bell pitch:  400    bell duration:  100
Pointer Control:
  acceleration:  20/10    threshold:  10
Screen Saver:
  prefer blanking:  no    allow exposures:  yes
  timeout:  0    cycle:  600
Colors:
  default colormap:  0x20    BlackPixel:  0x0    WhitePixel:  0xffff
Font Path:
  built-ins
DPMS (Energy Star):
  Standby: 600    Suspend: 600    Off: 600
  DPMS is Disabled
```

이렇게 스크린세이버 타임아웃 0, 에너지스타 DPMS Disabled 뜨면 성공이다. 그지같은 화면보호기를 걷어낼 수 있다.

`xscreensaver`와 같은것을 설치하고 xscreensaver 명령어를 사용해 끄는 방법도 있지만. 안그래도 모자란 용량 더 채우기 싫으니 이렇게 하자.


