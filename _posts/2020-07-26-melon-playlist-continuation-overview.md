---
title: "Melon Playlist Continuation - Overview"
date: 2020-07-26 19:20:00 +0900
tags: ["kakao arena", "melon playlist continuation"]
categories: [Kakao Arena, Melon Playlist Continuation]
toc: true
math: false
img_path: /assets/img/melon-playlist-continuation
---

![](melon.png){: width="150" height="150" }

[**Project Repository**](https://github.com/JehunYoo/Melon-Playlist-Continuation)

## Goal
주어진 playlist에 대해 **songs 100개**, **tags 10개**를 **순서에 맞게** 예측해야 한다.

Predict additional **100 songs** and **10 tags** **in orderly-correct way** for all given playlists.

## Data
대회에서 주어진 데이터는 크게 두가지 였다.
1. User data : List of playlists, Song meta data
2. Item data : Spectrogram of 707989 songs (about 232 GB)

Spectrogram은 용량이 너무 커서 사용하지 못했고 사용자 기반의 데이터만을 사용하였다.

## Models
다음 두가지 모델을 사용했다.
- K Nearest Neighbor
- Another Neighbor-based Model (Simply called it *Neighbor*.)

## EDA
### Group by Cases
`val.json`과 `test.json`의 각 playlist를 title, songs, tags의 유무로 분류하면 그 비율은 다음과 같다.

Title | Songs | Tags | val.json (%) | test.json (%)
:---:|:---:|:---:|---:|---:
X|X|X|0|0
X|X|O|0|0
X|O|X|42|42
X|O|O|39|39
O|X|X|8|8
O|X|O|11|11
O|O|X|0|0
O|O|O|0|0

[Reference](https://github.com/JehunYoo/Melon-Playlist-Continuation#references)에서 사용한 모델에 적용하기 위해 다음과 같이 case를 나눴다.

case:

1. songs & tags
1. songs only
1. tags only
1. title only = no songs & no tags

단, case 1, 2, 3에 대해 title은 있더라도 무시한다.

case 1에 대해서는 Neighbor을 사용한다.<br>
case 2, 3에 대해서는 Neighbor을 사용한 후에 KNN을 사용한다.<br>
case 4에 대해서는 title에서 미리 tag를 추출하여 case 1 or case 3으로 변형한다.

## Ideas
### Issue Date & Update Date
Candidate songs whose issue date is earlier than update date of the given playlist are excluded.<br>
From this, the model performed 0.0027 higher than before about total score.<br>
0.0027 improvement about total score is eqaul to 0.0039 improvement about song nDCG score.
