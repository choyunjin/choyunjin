# Insertion effect
Insertion effect는 지정된 채널에 갖가지 이펙트(distortion,overdrive 등등등 뭐가 많음)를 넣을 수 있는 기능으로, lrsynth에서는 1포트당 8개까지 사용 가능하며, 1번째 포트는 Roland GS sysex 로도 제어할 수 있음

# Sysex

## 기본 구조
```
f0 00 34 0f 11 21 rr rr rr vv ... f7
```
- `f0` = 이 메세지가 sysex 메세지임을 의미함
- `00 34 0f` = 생산자(manufacturer) ID (YJ)
- `00` = Device ID (사실상 무쓸모지만 가급적 00으로 놔두는 것을 권장)
- `11 21` = Model ID (lrsynth)
- `rr rr rr` = 파라미터 번호
- `vv ...` = 데이터(길이는 1바이트일 수도 있고 더 길 수도 있음)
- `f7` = sysex의 끝

## 이펙트 종류 설정
```
f0 00 34 0f 00 11 21 20 1x 00 vv vv vv f7
```
- `f0 00 34 0f 00 11 21` = (생략)
- `20 1x 00` = 파라미터 번호(이펙트 종류 설정)
  - `x` = 이펙터 번호(0~7)
- `vv vv vv` = 이펙트 종류
- `f7` = (생략)

## 채널별 이펙트 on/off
```
f0 00 34 0f 00 11 21 20 2x 0h vv f7
```
- `f0 00 34 0f 00 11 21` = (생략)
- `20 2x 0p` = 파라미터 번호(채널별 이펙트 on/off)
  - `x` = 이펙터 번호(0~7)
  - `h` = midi 채널 번호(0 = 10번 / 1,2,3,...,7,8,9 = 각각 1,2,3,...,7,8,9번 / a,b,c,d,e,f = 각각 11,12,13,14,15,16번)
- `vv` = 값(0 = off(bypass: 그냥 통과),1 = on)
- `f7` = (생략)

## 이펙트 파라미터
```
f0 00 34 0f 00 11 21 20 3x pp vv f7
```
- `f0 00 34 0f 00 11 21` = (생략)
- `20 3x pp` = 파라미터 번호(이펙트 파라미터)
  - `x` = 이펙터 번호(0~7)
  - `pp` = 파라미터 번호(00~2f인데 27부터는 어떤 이펙트를 선택하든 공통으로 나오는 파라미터임)
- `vv` = 파라미터 값
- `f7` = (생략)

# 이펙트 종류
뭐가 엄청 많지만 사실 별로 쓰는건 없어서 많이 쓰는 것부터 구현할 예정이다.
~취소선~이 쳐진 건 구현이 안 됐다는 뜻이다.

### None - `00 00 00`
기본값. 아무것도 적용하지 않는다.

## filter 계열

### Stereo EQ - `40 01 00`
4 band 이퀄라이저를 적용한다.

| 파라미터 번호 | 이름 | 값 | 기본값 | 설명 |
| --- | -------- | ----- | --- | ------------ |
| 0x01 | Low freq | N/A | N/A | 저음역대의 기준 주파수를 설정한다. |
| 0x02 | Low gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 저음역대의 gain을 조정한다. |
| 0x03 | High freq | N/A | N/A | 고음역대의 기준 주파수를 설정한다. |
| 0x04 | High gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 고음역대의 gain을 조정한다. |
| 0x14 | Level | 0 - 127<br>(0x00 - 0x7f) | 127 (0x7f) | 최종 출력 level을 조정한다. |

## distortion 계열
### Overdrive - `40 01 10`
일렉기타의 overdrive 이펙트를 적용한다.

| 파라미터 번호 | 이름 | 값 | 기본값 | 설명 |
| --- | -------- | ----- | --- | ------------ |
| 0x01 | Drive | 0 - 127<br>(0x00 - 0x7f) | 48 (0x30) | drive의 정도를 조정한다. |
| 0x02 | Amp type | Small,Built-in,2 stack,3 stack<br>(0x00,0x01,0x02,0x03) | Built-in (0x01) | 기타 앰프 종류를 선택한다. |
| 0x03 | Amp on/off | off,on<br>(0x00,0x01) | on (0x01) | 기타 앰프를 켜고 끈다. |
| 0x11 | EQ Low(200Hz) gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 저음역대의 EQ gain을 조정한다. |
| 0x12 | EQ High(4kHz) gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 고음역대의 EQ gain을 조정한다. |
| 0x13 | Pan | Auto - L63 - Center - R63<br>(0x00 - 0x01 - 0x40 - 0x7f) | Auto (0x00) | 이 값이 0 (0x00) 이 아닐 경우 채널 cc의 pan 설정값을 무시하고 여기서 설정한 pan 값에 강제로 맞춘다. |
| 0x14 | Level | 0 - 127<br>(0x00 - 0x7f) | 127 (0x7f) | 최종 출력 level을 조정한다. |

### Distortion - `40 01 11`
일렉기타의 distortion 이펙트를 적용한다.

| 파라미터 번호 | 이름 | 값 | 기본값 | 설명 |
| --- | -------- | ----- | --- | ------------ |
| 0x01 | Drive | 0 - 127<br>(0x00 - 0x7f) | 76 (0x4c) | drive의 정도를 조정한다. |
| 0x02 | Amp type | Small,Built-in,2 stack,3 stack<br>(0x00,0x01,0x02,0x03) | 3 stack (0x03) | 기타 앰프 종류를 선택한다. |
| 0x03 | Amp on/off | off,on<br>(0x00,0x01) | on (0x01) | 기타 앰프를 켜고 끈다. |
| 0x11 | EQ Low(200Hz) gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 저음역대의 EQ gain을 조정한다. |
| 0x12 | EQ High(4kHz) gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | -8 (0x38) | 고음역대의 EQ gain을 조정한다. |
| 0x13 | Pan | Auto - L63 - Center - R63<br>(0x00 - 0x01 - 0x40 - 0x7f) | Auto (0x00) | 이 값이 0 (0x00) 이 아닐 경우 채널 cc의 pan 설정값을 무시하고 여기서 설정한 pan 값에 강제로 맞춘다. |
| 0x14 | Level | 0 - 127<br>(0x00 - 0x7f) | 127 (0x7f) | 최종 출력 level을 조정한다. |

## modulation 계열

### ~Tremolo - `40 01 25`~
이름 그대로이다.

| 파라미터 번호 | 이름 | 값 | 기본값 | 설명 |
| --- | -------- | ----- | --- | ------------ |
| 0x01 | Mod wave | 0 - 4 | N/A | modulation의 종류를 설정한다.<br>0 = Triangle wave<br>1 = Square wave<br>2 = Sine wave<br>3 = Saw wave<br>4 = 뒤집어진 Saw wave |
| 0x02 | Mod rate | 0.05 - 10.0<br>(0x00 - 0x7f)[^Rate1] | N/A | modulation rate를 설정한다. |
| 0x03 | Mod depth | 0 - 127<br>(0x00 - 0x7f) | N/A | modulation depth를 설정한다. |
| 0x11 | EQ Low(200Hz) gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 저음역대의 EQ gain을 조정한다. |
| 0x12 | EQ High(4kHz) gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 고음역대의 EQ gain을 조정한다. |
| 0x14 | Level | 0 - 127<br>(0x00 - 0x7f) | 127 (0x7f) | 최종 출력 level을 조정한다. |

## compressor 계열

### ~Compressor - `40 01 30`~
일반적으로 알려진 컴프레서의 역할과 같다....만 뭔가 많이 다른 것 같다.

| 파라미터 번호 | 이름 | 값 | 기본값 | 설명 |
| --- | -------- | ----- | --- | ------------ |
| 0x01 | Attack | 0 - 127<br>(0x00 - 0x7f) | N/A | 입력되는 소리의 attack time을 설정한다. |
| 0x02 | Sustain | 0 - 127<br>(0x00 - 0x7f) | N/A | scva 설명서 봐라. |
| 0x03 | Post gain | 0,+6,+12,+18<br>(0x00 - 0x03) | N/A | 출력 gain을 조절한다. |
| 0x11 | EQ Low(200Hz) gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 저음역대의 EQ gain을 조정한다. |
| 0x12 | EQ High(4kHz) gain | -12 - +12<br>(0x34 - 0x40 - 0x4c) | 0 (0x40) | 고음역대의 EQ gain을 조정한다. |
| 0x13 | Pan | Auto - L63 - Center - R63<br>(0x00 - 0x01 - 0x40 - 0x7f) | Auto (0x00) | 이 값이 0 (0x00) 이 아닐 경우 채널 cc의 pan 설정값을 무시하고 여기서 설정한 pan 값에 강제로 맞춘다. |
| 0x14 | Level | 0 - 127<br>(0x00 - 0x7f) | 127 (0x7f) | 최종 출력 level을 조정한다. |

## chorus 계열

## delay,reverb 계열

## pitch shift 계열

## 기타

## 2개의 이펙터를 동시에 사용하는 것들

## 3개 이상의 이펙터를 동시에 사용하는 것들

## 2개의 이펙터가 좌,우 채널에 각각 적용되는 것들

### Drive 1 / 2 - `40 11 03`
Drive 계열 이펙트를 좌/우에 하나씩 적용한다.

| 파라미터 번호 | 이름 | 값 | 기본값 | 설명 |
| --- | -------- | ----- | --- | ------------ |
| 0x01 | Left:Type | Overdrive,Distortion<br>(0x00,0x01) | Overdrive (0x00) | 왼쪽 채널의 drive 이펙터 종류를 선택한다. |
| 0x02 | Left:Drive | 0 - 127<br>(0x00 - 0x7f) | 48 (0x30) | 왼쪽 채널의 drive의 정도를 조정한다. |
| 0x03 | Left:Amp type | Small,Built-in,2 stack,3 stack<br>(0x00,0x01,0x02,0x03) | Built-in (0x01) | 왼쪽 채널의 기타 앰프 종류를 선택한다. |
| 0x04 | Left:Amp on/off | off,on<br>(0x00,0x01) | on (0x01) | 왼쪽 채널의 기타 앰프를 켜고 끈다. |
| 0x06 | Right:Type | Overdrive,Distortion<br>(0x00,0x01) | Distortion (0x01) | 오른쪽 채널의 drive 이펙터 종류를 선택한다. |
| 0x07 | Right:Drive | 0 - 127<br>(0x00 - 0x7f) | 76 (0x4c) | 오른쪽 채널의 drive의 정도를 조정한다. |
| 0x08 | Right:Amp type | Small,Built-in,2 stack,3 stack<br>(0x00,0x01,0x02,0x03) | 3 stack (0x03) | 오른쪽 채널의 기타 앰프 종류를 선택한다. |
| 0x09 | Right:Amp on/off | off,on<br>(0x00,0x01) | on (0x01) | 오른쪽 채널의 기타 앰프를 켜고 끈다. |
| 0x10 | Left:Pan | L63 - Center - R63<br>(0x01 - 0x40 - 0x7f) | L63 (0x00) | 왼쪽 채널의 pan 값을 조절한다. |
| 0x11 | Left:Level | 0 - 127<br>(0x00 - 0x7f) | 96 (0x60) | 왼쪽 채널의 출력 level을 조정한다. |
| 0x12 | Right:Pan | L63 - Center - R63<br>(0x01 - 0x40 - 0x7f) | R63 (0x7f) | 오른쪽 채널의 pan 값을 조절한다. |
| 0x13 | Right:Level | 0 - 127<br>(0x00 - 0x7f) | 84 (0x54) | 오른쪽 채널의 출력 level을 조정한다. |
| 0x14 | Level | 0 - 127<br>(0x00 - 0x7f) | 127 (0x7f) | 최종 출력 level을 조정한다. |


[^Rate1]: 공식: x <= 0x63 일 때 0.05*x, 0x64 <= x <= 0x77 일 때 5+0.1(x - 99), 0x78 <= x <= 0x7d 일 때 7+0.5(x - 119), x >= 7e 일 때 10
