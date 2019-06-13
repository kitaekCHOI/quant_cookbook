# 데이터 정리하기

앞장에서는 API와 크롤링을 통하여 퀀트 투자에 필요한 주가, 재무제표, 가치지표를 수집하는 방법에 대해 배웠습니다. 이번 장에서는 각각 csv 파일로 저장된 데이터들을 하나로 정리하는 과정을 살펴보도록 하겠습니다.

## 주가 정리하기

주가의 경우 **data/KOR_price** 폴더 내 **티커.csv** 파일로 저장되어 있습니다. 해당 파일들을 불러온 후 데이터를 묶는 작업을 통해 하나의 파일로 합치는 방법에 대해 알아보도록 하겠습니다.


```r
library(stringr)
library(xts)
library(magrittr)

KOR_ticker = read.csv('data/KOR_ticker.csv', row.names = 1)
KOR_ticker$'종목코드' = str_pad(KOR_ticker$'종목코드', 6, side = c('left'), pad = '0')

price_list = list()
for (i in 1 : nrow(KOR_ticker)) {
  
  name = KOR_ticker[i, '종목코드']
  price_list[[i]] = read.csv(paste0('data/KOR_price/', name, '_price.csv'), row.names = 1) %>%
    as.xts()
  
}

price_list = do.call(cbind, price_list) %>% na.locf()
colnames(price_list) = KOR_ticker$'종목코드'

head(price_list[, 1:5])
```

```
##            005930 000660 005380 068270 051910
## 2017-05-23     NA     NA     NA  91678 289500
## 2017-05-24     NA     NA 164000  93149 290000
## 2017-05-25     NA     NA 165000  91972 296000
## 2017-05-26     NA     NA 163500  91972 304500
## 2017-05-29  45620  57900 162000  91776 309500
## 2017-05-30  44640  57400 164000  93933 306500
```

1. 먼저 티커가 저장된 csv 파일을 불러온 후, 티커를 6자리로 맞춰주도록 합니다. 
2. 빈 리스트인 pirce_list를 생성합니다.
3. for loop 구문을 이용해 종목별 가격 데이터를 불러온 후, `as.xts()`를 통해 시계열 형태로 데이터를 변경한 후 리스트에 저장합니다.
4. `do.call()` 함수를 통해 리스트를 열의 형태로 묶습니다.
5. 간혹 결측치가 발생할 수 있으므로, `na.locf()` 함수를 통해 결측치의 경우 전일 데이터를 사용하도록 합니다.
6. 행 이름을 각 종목의 티커로 변경해 주도록 합니다.

해당 작업을 통해 각각 흩어져있던 가격 데이터가 하나의 데이터로 잘 묶어지게 되었습니다.



```r
write.csv(data.frame(price_list), 'data/KOR_price.csv')
```

마지막으로 해당 데이터를 data 폴더 내 **KOR_price.csv** 파일로 저장해주도록 합니다. 시계열 형태 그대로 저장할 경우 인덱스가 삭제되므로, 데이터프레임 형태로 변경한 후 저장하도록 합니다.

## 재무제표 정리하기
