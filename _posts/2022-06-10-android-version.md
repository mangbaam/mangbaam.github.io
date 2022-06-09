---
layout: post
title: 내가 보려고 만든 안드로이드 버전 + API Level
subtitle: Android version + API level + Codename
categories: Android
tags: [version]
---

## ⭐

안드로이드를 개발하다 보면 현재 최신 안드로이드 버전이 무엇이고, 어떤 라이브러리는 타겟 SDK가 몇이고 킷캣이 어쩌고 오레오가 어쩌고... 정리가 안 된 상태로 혼란을 겪고 있다가 안드로이드 개발 공부한다고 하면 최소 버전 정보는 알고 있어야겠다는 생각이 들어 이번 기회에 정리하게 되었다.

## 표 보는 법

- [누적 사용량](https://gs.statcounter.com/android-version-market-share/mobile-tablet/worldwide)은 2022년 5월 31일에 업데이트 된 데이터이다.
- Android 10부터 Google이 공개적으로 코드명을 사용하는 것을 중단했기 때문에 아래 표에 Android 10 이상의 코드명은 내부 코드명이다.
- 코드 네임은 Android 1.5 Cupcake가 출시되면서 만들어졌기 때문에 그 이전의 코드 네임은 공식 명칭이 아닐 수 있다.
- 아래 표의 출처: [https://apilevels.com/](https://apilevels.com/)

## 한 눈에 보기

<table class="full-width">
  <tbody>
    <tr>
      <th>버전</th>
      <th>SDK / API level</th>
      <th>
        <a
          href="https://developer.android.com/reference/kotlin/android/os/Build.VERSION_CODES"
          >버전 코드</a
        >
      </th>
      <th>코드 네임</th>
      <th>누적 사용량</th>
      <th>Year</th>
    </tr>
    <tr>
      <td>
        <b
          ><a href="https://developer.android.com/about/versions/13"
            >Android 13</a
          ></b
        >
        <sup class="beta">BETA</sup>
      </td>
      <td>Level 33</td>
      <td><code>T</code></td>
      <td>
        티라미수<br />
        (Tiramisu)
      </td>
      <td rowspan="2"><i>No data</i></td>
      <td rowspan="2"><i>TBA</i></td>
    </tr>
    <tr>
      <td rowspan="3">
        <b
          ><a href="https://developer.android.com/about/versions/12"
            >Android 12</a
          ></b
        >
      </td>
      <td>
        Level 32
        <span class="subversion">Android 12L</span>
      </td>
      <td><code>S_V2</code></td>
      <td rowspan="2">
        스노우 콘<br />
        (Snow Cone)
      </td>
    </tr>
    <tr>
      <td>Level 31 <span class="subversion">Android 12</span></td>
      <td><code>S</code></td>
      <style>
        #cell146 {
          width: 100%;
          height: 100%;
          background: linear-gradient(to right, #afe4ae 14.6%, white 14.6%);
        }
        @media (prefers-color-scheme: light) {
          #cell146 {
            background: linear-gradient(
              to right,
              #006236 14.6%,
              rgba(0, 0, 0, 0) 14.6%
            );
          }
        }
      </style>
      <td rowspan="2" id="cell146">14.6%</td>
      <td rowspan="2">2021</td>
    </tr>
    <tr class="table-notes">
      <td colspan="3">
        <ul>
          <li>
            2022년 8월부터 <i>신규 앱</i>은 <code>targetSdk</code>
            <a
              href="https://developer.android.com/google/play/requirements/target-sdk"
              >31 이상</a
            >을 타겟팅하고 변경사항에 맞게 조정해야 한다.
          </li>
          <li>
            2023년 11월까지 <i>기존의 모든 앱</i>은 <code>targetSdk</code>
            <a
              href="https://support.google.com/googleplay/android-developer/answer/11926878"
              >31 이상</a
            >을 타겟팅 해야한다.
          </li>
        </ul>
        <tr>
          <td rowspan="2">
            <b
              ><a href="https://developer.android.com/about/versions/11"
                >Android 11</a
              ></b
            >
          </td>
          <td>Level 30</td>
          <td><code>R</code></td>
          <td>
            레드벨벳 케이크<br />
            (Red Velvet Cake)
          </td>
          <style>
            #cell477 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 47.7%, white 47.7%);
            }
            @media (prefers-color-scheme: light) {
              #cell477 {
                background: linear-gradient(
                  to right,
                  #006236 47.7%,
                  rgba(0, 0, 0, 0) 47.7%
                );
              }
            }
          </style>
          <td rowspan="2" id="cell477">47.7%</td>
          <td rowspan="2">2020</td>
        </tr>
        <tr class="table-notes">
          <td colspan="3">
            <ul>
              <li>
                현재 <i>신규 앱</i>은 <code>targetSdk</code>
                <a
                  href="https://developer.android.com/google/play/requirements/target-sdk"
                  >30 이상</a
                >을 타겟팅하고 변경사항에 맞게 조정해야 한다.
              </li>
              <li>
                2022년 11월까지 <i>기존의 모든 앱</i>은 <code>targetSdk</code>
                <a
                  href="https://support.google.com/googleplay/android-developer/answer/11926878"
                  >30 이상</a
                >을 타겟팅 해야한다.
              </li>
            </ul>
          </td>
        </tr>
        <tr>
          <td>
            <b
              ><a href="https://developer.android.com/about/versions/10"
                >Android 10</a
              ></b
            >
          </td>
          <td>Level 29</td>
          <td><code>Q</code></td>
          <td>
            퀸스 타르트<br />
            (Quince Tart)
          </td>
          <style>
            #cell703 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 70.3%, white 70.3%);
            }
            @media (prefers-color-scheme: light) {
              #cell703 {
                background: linear-gradient(
                  to right,
                  #006236 70.3%,
                  rgba(0, 0, 0, 0) 70.3%
                );
              }
            }
          </style>
          <td rowspan="1" id="cell703">70.3%</td>
          <td>2019</td>
        </tr>
        <tr>
          <td rowspan="2">
            <b
              ><a href="https://developer.android.com/about/versions/pie"
                >Android 9</a
              ></b
            >
          </td>
          <td>Level 28</td>
          <td><code>P</code></td>
          <td>파이<br />(Pie)</td>
          <style>
            #cell816 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 81.6%, white 81.6%);
            }
            @media (prefers-color-scheme: light) {
              #cell816 {
                background: linear-gradient(
                  to right,
                  #006236 81.6%,
                  rgba(0, 0, 0, 0) 81.6%
                );
              }
            }
          </style>
          <td rowspan="2" id="cell816">81.6%</td>
          <td rowspan="2">2018</td>
        </tr>
        <tr class="table-notes">
          <td colspan="3">
            <ul>
              <li>
                신규 Wear OS 및 Wear OS 업데이트의 <code>targetSdk</code>는
                <a
                  href="https://developer.android.com/google/play/requirements/target-sdk"
                  >28 이상</a
                >이어야 한다.
              </li>
            </ul>
          </td>
        </tr>
        <tr>
          <td rowspan="2">
            <b
              ><a href="https://developer.android.com/about/versions/oreo"
                >Android 8</a
              ></b
            >
          </td>
          <td>Level 27 <span class="subversion">Android 8.1</span></td>
          <td><code>O_MR1</code></td>
          <td rowspan="2">오레오<br />(Oreo)</td>
          <style>
            #cell875 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 87.5%, white 87.5%);
            }
            @media (prefers-color-scheme: light) {
              #cell875 {
                background: linear-gradient(
                  to right,
                  #006236 87.5%,
                  rgba(0, 0, 0, 0) 87.5%
                );
              }
            }
          </style>
          <td rowspan="1" id="cell875">87.5%</td>
          <td rowspan="2">2017</td>
        </tr>
        <tr>
          <td>Level 26 <span class="subversion">Android 8.0</span></td>
          <td><code>O</code></td>
          <style>
            #cell902 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 90.2%, white 90.2%);
            }
            @media (prefers-color-scheme: light) {
              #cell902 {
                background: linear-gradient(
                  to right,
                  #006236 90.2%,
                  rgba(0, 0, 0, 0) 90.2%
                );
              }
            }
          </style>
          <td rowspan="1" id="cell902">90.2%</td>
        </tr>
        <tr>
          <td rowspan="2">
            <b
              ><a href="https://developer.android.com/about/versions/nougat"
                >Android 7</a
              ></b
            >
          </td>
          <td>Level 25 <span class="subversion">Android 7.1</span></td>
          <td><code>N_MR1</code></td>
          <td rowspan="2">누가<br />(Nougat)</td>
          <style>
            #cell918 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 91.8%, white 91.8%);
            }
            @media (prefers-color-scheme: light) {
              #cell918 {
                background: linear-gradient(
                  to right,
                  #006236 91.8%,
                  rgba(0, 0, 0, 0) 91.8%
                );
              }
            }
          </style>
          <td rowspan="1" id="cell918">91.8%</td>
          <td rowspan="2">2016</td>
        </tr>
        <tr>
          <td>Level 24 <span class="subversion">Android 7.0</span></td>
          <td><code>N</code></td>
          <style>
            #cell946 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 94.6%, white 94.6%);
            }
            @media (prefers-color-scheme: light) {
              #cell946 {
                background: linear-gradient(
                  to right,
                  #006236 94.6%,
                  rgba(0, 0, 0, 0) 94.6%
                );
              }
            }
          </style>
          <td rowspan="1" id="cell946">94.6%</td>
        </tr>
        <tr>
          <td>
            <b
              ><a
                href="https://developer.android.com/about/versions/marshmallow"
                >Android 6</a
              ></b
            >
          </td>
          <td>Level 23</td>
          <td><code>M</code></td>
          <td>마시멜로우<br />(Marshmallow)</td>
          <style>
            #cell971 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 97.1%, white 97.1%);
            }
            @media (prefers-color-scheme: light) {
              #cell971 {
                background: linear-gradient(
                  to right,
                  #006236 97.1%,
                  rgba(0, 0, 0, 0) 97.1%
                );
              }
            }
          </style>
          <td rowspan="1" id="cell971">97.1%</td>
          <td>2015</td>
        </tr>
        <tr>
          <td rowspan="3">
            <b
              ><a href="https://developer.android.com/about/versions/lollipop"
                >Android 5</a
              ></b
            >
          </td>
          <td>Level 22 <span class="subversion">Android 5.1</span></td>
          <td><code>LOLLIPOP_MR1</code></td>
          <td rowspan="2">롤리팝<br />(Lollipop)</td>
          <style>
            #cell987 {
              width: 100%;
              height: 100%;
              background: linear-gradient(to right, #afe4ae 98.7%, white 98.7%);
            }
            @media (prefers-color-scheme: light) {
              #cell987 {
                background: linear-gradient(
                  to right,
                  #006236 98.7%,
                  rgba(0, 0, 0, 0) 98.7%
                );
              }
            }
          </style>
          <td rowspan="1" id="cell987">98.7%</td>
          <td>2015</td>
        </tr>
        <tr>
          <td>Level 21 <span class="subversion">Android 5.0</span></td>
          <td><code>LOLLIPOP</code>, <code>L</code></td>
          <td rowspan="24"><i>No data</i></td>
          <td rowspan="3">2014</td>
        </tr>
        <tr class="table-notes">
          <td colspan="3">
            <ul>
              <li>
                <a href="https://developer.android.com/jetpack/compose"
                  >Jetpack Compose</a
                >의 <code>minSdk</code>는 21 이상이다.
              </li>
            </ul>
          </td>
        </tr>
        <tr>
          <td rowspan="9"><b>Android 4</b></td>
          <td>
            Level 20
            <span class="subversion">Android 4.4W</span>
          </td>
          <td><code>KITKAT_WATCH</code></td>
          <td rowspan="2">킷캣<br />(KitKat)</td>
        </tr>
        <tr>
          <td>
            Level 19
            <span class="subversion">Android 4.4</span>
          </td>
          <td><code>KITKAT</code></td>
          <td rowspan="3">2013</td>
        </tr>
        <tr class="table-notes">
          <td colspan="3">
            <ul>
              <li>
                Google Play는 API level 19 미만을
                <a
                  href="https://android-developers.googleblog.com/2021/07/google-play-services-discontinuing-jelly-bean.html"
                  >지원하지 않는다.</a
                >
              </li>
            </ul>
          </td>
        </tr>
        <tr>
          <td>Level 18 <span class="subversion">Android 4.3</span></td>
          <td><code>JELLYBEAN_MR2</code></td>
          <td rowspan="3">젤리빈<br />(Jelly Bean)</td>
        </tr>
        <tr>
          <td>Level 17 <span class="subversion">Android 4.2</span></td>
          <td><code>JELLYBEAN_MR1</code></td>
          <td rowspan="2">2012</td>
        </tr>
        <tr>
          <td>Level 16 <span class="subversion">Android 4.1</span></td>
          <td><code>JELLYBEAN</code></td>
        </tr>
        <tr>
          <td>
            Level 15 <span class="subversion">Android 4.0.3 – 4.0.4</span>
          </td>
          <td><code>ICE_CREAM_SANDWICH_MR1</code></td>
          <td rowspan="2">아이스크림 샌드위치<br />(Ice Cream Sandwich)</td>
          <td rowspan="7">2011</td>
        </tr>
        <tr>
          <td>
            Level 14 <span class="subversion">Android 4.0.1 – 4.0.2</span>
          </td>
          <td><code>ICE_CREAM_SANDWICH</code></td>
        </tr>
        <tr class="table-notes">
          <td colspan="3">
            <ul>
              <li>
                <a href="https://developer.android.com/jetpack">Jetpack</a>/<a
                  href="https://developer.android.com/jetpack/androidx"
                  >AndroidX</a
                >
                라이브러리는 <code>minSdk</code> 14 이상을
                <a
                  href="https://developer.android.com/topic/libraries/support-library#api-versions"
                  >요구한다</a
                >
              </li>
            </ul>
          </td>
        </tr>
        <tr>
          <td rowspan="3"><b>Android 3</b></td>
          <td>Level 13 <span class="subversion">Android 3.2</span></td>
          <td><code>HONEYCOMB_MR2</code></td>
          <td rowspan="3">허니콤<br />(Honeycomb)</td>
        </tr>
        <tr>
          <td>Level 12 <span class="subversion">Android 3.1</span></td>
          <td><code>HONEYCOMB_MR1</code></td>
        </tr>
        <tr>
          <td>Level 11 <span class="subversion">Android 3.0</span></td>
          <td><code>HONEYCOMB</code></td>
        </tr>
        <tr>
          <td rowspan="6"><b>Android 2</b></td>
          <td>
            Level 10 <span class="subversion">Android 2.3.3 – 2.3.7</span>
          </td>
          <td><code>GINGERBREAD_MR1</code></td>
          <td rowspan="2">진저브레드<br />(Gingerbread)</td>
        </tr>
        <tr>
          <td>Level 9 <span class="subversion">Android 2.3.0 – 2.3.2</span></td>
          <td><code>GINGERBREAD</code></td>
          <td rowspan="3">2010</td>
        </tr>
        <tr>
          <td>Level 8 <span class="subversion">Android 2.2</span></td>
          <td><code>FROYO</code></td>
          <td>프로요<br />(Froyo)</td>
        </tr>
        <tr>
          <td>Level 7 <span class="subversion">Android 2.1</span></td>
          <td><code>ECLAIR_MR1</code></td>
          <td rowspan="3">
            에클레어
            <br />
            (Eclair)
          </td>
        </tr>
        <tr>
          <td>Level 6 <span class="subversion">Android 2.0.1</span></td>
          <td><code>ECLAIR_0_1</code></td>
          <td rowspan="5">2009</td>
        </tr>
        <tr>
          <td>Level 5 <span class="subversion">Android 2.0</span></td>
          <td><code>ECLAIR</code></td>
        </tr>
        <tr>
          <td rowspan="4"><b>Android 1</b></td>
          <td>Level 4 <span class="subversion">Android 1.6</span></td>
          <td><code>DONUT</code></td>
          <td>도넛<br />(Donut)</td>
        </tr>
        <tr>
          <td>Level 3 <span class="subversion">Android 1.5</span></td>
          <td><code>CUPCAKE</code></td>
          <td>컵케이크<br />(Cupcake)</td>
        </tr>
        <tr>
          <td>Level 2 <span class="subversion">Android 1.1</span></td>
          <td><code>BASE_1_1</code></td>
          <td>쁘띠 푸르<br />(Petit Four)</td>
        </tr>
        <tr>
          <td>Level 1 <span class="subversion">Android 1.0</span></td>
          <td><code>BASE</code></td>
          <td>
            <i>애플파이<br />(Apple Pie)</i>
          </td>
          <td>2008</td>
        </tr>
      </td>
    </tr>
  </tbody>
</table>

## 버전 별 특징

### SDK 14

- Jetpack과 AndroidX 라이브러리가 지원되는 최소 버전이다. 그렇기 때문에 사실상 이 이하의 버전은 다루지 않을 가능성이 높다.

### SDK 19

- Google Play가 지원되는 최소 버전이다.

### SDK 21

- Jetpack Compose를 지원하는 최소 버전이다.
- [카카오뱅크](https://m.kakaobank.com/Notices/view/12112)가 지원하는 최소 버전이다.

### SDK 22

- SDK 22 이상의 버전을 사용하는 사용자가 전체의 98.7%이다.

### SDK 23

- 카카오톡이 지원하는 최소 버전이다.
- [카카오페이](https://www.kakaopay.com/news/notice_detail?id=1435&page=1)가 지원하는 최소 버전이다.

### SDK 28

- 새로 만드는 Wear OS나 Wear OS 업데이트를 위해서 `targetSdk`는 28 이상이어야 한다.

### SDK 31

- 현재 새로 만드는 앱은 `targetSdk`를 31 이상으로 해야한다. [링크](https://developer.android.com/google/play/requirements/target-sdk)
- 기존의 앱들도 2022년 11월까지 `targetSdk`를 30 이상으로 올려야 한다. [링크](https://support.google.com/googleplay/android-developer/answer/11926878)

### SDK 32

- 2022년 8월 부터는 새로 만드는 앱은 `targetSdk`를 31 이상으로 해야한다.[링크](https://developer.android.com/google/play/requirements/target-sdk)
- 기존의 앱들은 2023년 11월까지 `targetSdk`가 31 이상이 되도록 해야한다.[링크](https://support.google.com/googleplay/android-developer/answer/11926878)
