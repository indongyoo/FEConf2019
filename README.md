# FEConf2019 - ES6+ 비동기 프로그래밍과 실전 에러 핸들링

## 이미지 가져오기

```javascript
const {log, clear} = console;

const imgs = [
  { name: "HEART", url: "https://s3.marpple.co/files/m2/t3/colored_images/45_1115570_1162087_150x0.png" },
  { name: "6", url: "https://s3.marpple.co/f1/2018/1/1054966_1516076919028_64501_150x0.png"},
  { name: "하트", url: "https://s3.marpple.co/f1/2019/1/1235206_1548918825999_78819_150x0.png" },
  { name: "도넛", url:"https://s3.marpple.co/f1/2019/1/1235206_1548918758054_55883_150x0.png"},
];

const imgs2 = [
  { name: "HEART", url: "https://s3.marpple.co/files/m2/t3/colored_images/45_1115570_1162087_150x0.png" },
  { name: "6", url: "https://s3.marpple.co/f1/2018/1/1054966_1516076919028_64501_150x0.jpg"},
  { name: "하트", url: "https://s3.marpple.co/f1/2019/1/1235206_1548918825999_78819_150x0.png" },
  { name: "도넛", url:"https://s3.marpple.co/f1/2019/1/1235206_1548918758054_55883_150x0.png"},
];

const loadImage = url => new Promise((resolve, reject) => {
  let img = new Image();
  img.src = url;
  // log('이미지로드: ', url);
  img.onload = function() {
    resolve(img);
  };
  img.onerror = function(e) {
    reject(e);
  };
  return img;
});

// loadImage(imgs[0].url).then(img => log(img.height));

```

## 실전
- 이미지들을 불러와서 모든 이미지의 높이를 더한다.

```javascript
async function f1() {
  try {
    let error = null;
    const total = await imgs2
      .map(async ({url}) => {
        if (error) return;
        try {
          const img = await loadImage(url);
          return img.height;
        } catch (e) {
          log(e);
          throw e;
        }
      })
      .reduce(async (total, height) => await total + await height, 0);

    log(total);
  } catch (e) {
    log(0);
  }
}
// f1();
```
- 위 코드는 에러를 어설프게 핸들링하여 오히려 에러보다 심한 버그가 생긴 코드입니다.

## 해결
- Promise, async/await, try/catch 잘 다루기

```javascript
function* map(f, iter) {
  for (const a of iter) {
    yield a instanceof Promise ? a.then(f) : f(a);
  }
}
async function reduceAsync(f, acc, iter) {
  for await (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
}

const f2 = imgs =>
  reduceAsync((a, b) => a + b, 0,
    map(img => img.height,
      map(({url}) => loadImage(url), imgs)));

// f2(imgs).catch(_ => 0).then(log);
f2(imgs2).catch(_ => 0).then(log);
```

## 정리

- Promise, async/await, try/catch를 정확히 다루는 것이 중요합니다.
- 제너레이터/이터레이터/이터러블을 잘 응용하면 코드의 표현력을 더할 뿐 아니라 에러 핸들링도 더 잘할 수 있습니다.
- 순수 함수에서는 에러가 발생되도록 그냥 두는 것이 더 좋습니다.
- 에러 핸들링 코드는 부수효과를 일으킬 코드 주변에 작성하는 것이 좋습니다.
- 불필요하게 에러 핸들링을 미리 해두는 것은 에러를 숨길 뿐입니다.
- 차라리 에러를 발생시키는게 낫습니다. sentry.io 같은 서비스를 이용하여 발생되는 모든 에러를 볼 수 있도록 하는 것이 고객과 회사를 위하는 더 좋은 해법입니다.
