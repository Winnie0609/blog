---
title: React Movie App 實作
date: 2021-04-17 22:51:49
tags: react
description: 使用 React 實作 Movie App. 這裡練習了如何使用 TMDB 提供的 API 來獲取需要的資料、使用 useState 以及 useEffect 來更新資料、拆分 component、使用 styled component 的方法。
---

[Github](https://github.com/Winnie0609/movie-app)  
[Live Demo](https://winnie0609.github.io/movie-app/)

[![Search Movie App Demo](https://i.imgur.com/3nQ48gx.gif)](https://i.imgur.com/3nQ48gx.gif)

## 簡介

這個為了練習使用 React 的邏輯以及編寫方式所刻的 Movie App。這裡練習了如何使用 TMDB 提供的 API 來獲取需要的資料、使用 useState 以及 useEffect 來更新資料、拆分 component、使用 styled component 的方法。下面記錄了搜尋頁面 fetch API 的詳細步驟，其他頁面則只是記錄重點功能。參考資料附在文末。

另外，因為專案一直在添加新的功能，因此檔案名字以及 Code 都會有調整，並不完全跟筆記的一樣。

## 功能

- 首頁有即將上映之電影 & 熱門電影/電視劇 (可通過點擊按鈕切換)
- 電影的呈現方式使用橫軸的滾動條
- 電影頁則是以卡片的形式呈現，點擊卡片會出現電影資訊
- 彈出的 Modal 內含有電影預告的網址，導致外部鏈接
- 搜尋到的電影可以加入 favourite list 裡，同時也會加到 local storage 裡
- 已經儲存的電影可點擊菜單列的愛心查看
- 無法顯示的照片使用默認照片顯示

## 前置作業

- 需要使用的 API : [The Movie Database API](https://www.themoviedb.org/)
- 先到 TMDB 網站申請賬號，再根據 [官方文件](https://developers.themoviedb.org/3/getting-started/introduction) 申請網站的 API key .文件上都有清楚的申請步驟，申請成功後，把 key 以及 Access Token 保存下來。
- Modal 以及 Pagination 使用 meterial ui

```bash
$ npm install @material-ui/lab
```

## 步驟

### 搜尋頁面

#### 設置基本結構以及樣式

簡單設置需要使用到的 class component ，在內添加 JRX 以及引入外部 CSS file 用以測試文件是否能夠渲染到瀏覽器上。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

class Main extends React.Component {
  render() {
    return (
      <div className="container">
        <header>
          <h1 className="title"> Search a movie.</h1>
        </header>
      </div>
    );
  }
}

ReactDOM.render(<Main />, document.getElementById('root'));
```

#### 創造一個 component

使用 form 創造一個 searchMovie component ，裡面需要有 input, label 以及 submit button. 在文件的結尾 export 後，再於 index.js file 中引入 App.js.

```js
function Search() {
  return (
    <form className="form">
      <label htmlFor="query">Movie name</label>
      <input type="text" name="query" placeholder="Goziila vs Kong..." />
      <button className="submitBtn" type="submit">
        submit
      </button>
    </form>
  );
}
```

[![](https://i.imgur.com/QwKrXVI.png)](https://i.imgur.com/QwKrXVI.png)

#### 透過 API 抓取需要的資料

首先創造 searchMovies function，在 button 添加 onSubmit attribute 指向剛才創造的 function. 這個 function 要先將原本 onSubmit 預設的動作清除 (preventDefault) 加上我們想要的操作。在加上需要執行的代碼前，可以先簡單測試是否成功將預設的動作清除。

API 的組成如下，當中的搜索方式，TMDB 提供了了三種：search / discover /find，各有不同的功能，詳細的可以查看[官方文件](https://www.themoviedb.org/documentation/api)。

[![](https://i.imgur.com/NCtZOrY.png)](https://i.imgur.com/NCtZOrY.png)

除了成功取得資料外，亦要考慮無法獲得資料的情況。因此可以使用 `try` `catch` 來做 error handling.在還沒獲取用戶輸入的資料前，可以先設置 query 來測試 API 是否運作順利。

```js
const API_KEY = process.env.REACT_APP_API_KEY;
const searchMovies = async (e) => {
  e.preventDefault();
  console.log('submitting');

  const query = 'Jurassic Park'; //測試用，query應是用戶輸入的資料

  const url = `https://api.themoviedb.org/3/search/movie/?api_key=${API_KEY}&language=en-US&query=${query}`;

  try {
    //成功的情況
    const res = await fetch(url);
    const data = await res.json();
    console.log(data);
  } catch (err) {
    //失敗的情況
    console.error(err);
  }
};
```

成功獲得資料的結果如下，搜尋 “Jurassic Park” 會出現 15 筆資料。我們需要的資料存在 results 裡。

[![](https://i.imgur.com/d7U50l3.png)](https://i.imgur.com/d7U50l3.png)

#### 使用 useState 更新 query

在 input 內添加 `value` , `value` 為 {query}，即是用戶輸入的資訊。使用 `useState` 和`useEffect` 來更新 value 的值。`const [query, setQuery] = useState("")` query 為`value` 變數；setQuery 用來更新 query 的值；`useState` 內的則是 query 的初始值。在 input 中添加 `onChange`，使用 `setQuery` 更新 `value` 。

```js
const [query, setQuery] = useState('');

<input
  type="text"
  name="query"
  placeholder="Goziila vs Kong..."
  value={query} //添加value
  onChange={(e) => setQuery(e.target.value)} //更新value
/>;
```

[![](https://i.imgur.com/v5sLqzt.gif)](https://i.imgur.com/v5sLqzt.gif)

#### 使用 useState, useEffect 更新 movie (顯示給用戶的資料)

獲得搜尋資料之後，需要將這些資料渲染到瀏覽器，因此需要一個 array 來存取這些資料。一樣先使用 useState 要設置 movies array，再用 useEffect 更新。把 movies array 更新成搜尋後獲得的資料。

```js
const [movies, setMovies] = useState([]); //movies是個array

try {
  const res = await fetch(url);
  const data = await res.json();
  setMovies(data.results); //把 movies array 更新成搜尋後獲得的資料
} catch (err) {
  console.error(err);
}
```

#### 把搜尋結果呈現在畫面上

[![](https://i.imgur.com/5Kt9Edt.png)](https://i.imgur.com/5Kt9Edt.png)

需要顯示在畫面上的資訊有：海報（poster_path），發行日期（release_date），電影簡介（overview），評分（vote_average）。可以用 movies array（裡面有剛才存進去的資料） 中調用我們需要的資料。使用 map 遍歷每一個搜尋結果，調用需要的資訊。

```js
<div className="card-list">
  {movies.map(movie => (
    <div className="card-info" key={movie.id}> //使用map都要加上key
      <h3>{movie.title}</h3> //標題
      <p>Release date: {movie.release\_date}</p> //發行日期
      <p>OverView: {movie.overview}</p> //簡介
      <p>Rating: {movie.vote\_average}</p> //評分
    </div>
  ))}
</div>
```

[![](https://i.imgur.com/FS4E4fj.png)](https://i.imgur.com/FS4E4fj.png)

海報照片有固定的 url 格式，改變的只有結尾的 poster_path ，因此只要更換最後這個部分就可以獲得海報連接。某些原本就沒有海報照片的電影會無法顯示，可以使用兩種方法解決這個狀況：**直接篩掉沒有海報的電影**或是**顯示代替圖案**。

```js
https://image.tmdb.org/t/p/[width size]/[poster_path]
```

方法 1 : 直接篩掉沒有海報的電影 (.fliter)

```js
<div className="card-list">
  //使用 .filter 只抓取 poster\_path 為 true 的電影
  {movies
    .filter((movie) => movie.poster_path)
    .map((movie) => (
      <div className="card-info" key={movie.id}>
        <img
          src={`https://image.tmdb.org/t/p/w300/${movie.poster_path}`}
          alt={movie.title}
        />
      </div>
    ))}
</div>
```

方法 2 : 顯示代替圖案 (onError)

```js
<div className="card-list">
  {movies.map((movie) => (
    <div className="card-info" key={movie.id}>
      <img
        src={`https://image.tmdb.org/t/p/w300/${movie.poster_path}`}
        alt={movie.title}
        //當url為null時，使用替代圖片
        onError={(e) => {
          e.target.onerror = null;
          e.target.src = 'https://i.postimg.cc/3RpfrHDh/photo.png';
        }}
      />
    </div>
  ))}
</div>
```

[![](https://i.imgur.com/mqd2xl4.png)](https://i.imgur.com/mqd2xl4.png)

### 多個頁面

Movies 頁面抓取 10 頁 Top rated 的電影。這裡頁面切換使用 material ui 的 Pagination 製作。在抓取需要的資料後，設置 page 的 state，默認為 1，即抓取第一頁。

```js
import React, { useState, useEffect } from 'react';
import { Container, MovieCard } from '../AllMovies/AllMovie_styles';
import CustomPagination from './CustomPagination';
import ContentModal from '../ContentModal/ContentModal';

function AllMovies() {
  const [AllMovies, setAllMovies] = useState([]);
  const [page, setPage] = useState(1);

  const API_KEY = process.env.REACT_APP_API_KEY;
  const API_URL = 'https://api.themoviedb.org/3';
  const all_movie_url = `${API_URL}/movie/top_rated?api_key=${API_KEY}&language=en-US&page=${page}`;

  async function fetchAllMovies() {
    const res = await fetch(all_movie_url);
    const data = await res.json();
    setAllMovies(data);
    console.log(AllMovies.results);
  }

  useEffect(() => {
    fetchAllMovies();
    window.scroll(0, 0); //回到最頂端
  }, [page]);

  return (
    <>
      <Container>
        <h2>What to watch</h2>
        {AllMovies.results &&
          AllMovies.results.map((movie) => (
            //顯示的資料
            <MovieCard>. . .</MovieCard>
          ))}
        //傳入 CustomPagination component
        <CustomPagination setPage={setPage} />
      </Container>
    </>
  );
}

export default AllMovies;
```

在 Pagination component 裡處理頁面更新 page 的 state，將 page 更換成點擊的頁數，資料便會根據該頁數抓取那頁的資料。

```js
import React from 'react';
import Pagination from '@material-ui/lab/Pagination';
import styled from 'styled-components';

function CustomPagination({ setPage, numberOfPages = 10 }) {
  const handlePageChange = (page) => {
    setPage(page);
    window.scroll(0, 0);
  };

  return (
    <Pagination
      onChange={(e) => handlePageChange(e.target.textContent)}
      count={numberOfPages} //會顯示幾頁
      shape="rounded"
      hideNextButton
      hidePrevButton
    />
  );
}

export default CustomPagination;
```

[![](https://i.imgur.com/HztIG7y.gif)](https://i.imgur.com/HztIG7y.gif)

## 小結

Search 頁面是第一個做的頁面，因此記錄得比較詳細，其他部分只記錄了一些沒有用過的功能。Live Demo 中無法點擊的部分是還沒做好的功能，裡面也還有 bug 還沒修好。

由於個別頁面和功能都是獨立做的，因此相似的功能會重複寫，架構也不是那麼清楚。之後有時間會把它再重新整理一下，筆記也會再同步更新。

## 參考資料

1. [Learn React in 1 Hour by Building a Movie Search App](https://www.freecodecamp.org/news/learn-react-in-1-hour-by-building-a-movie-search-app/)  
2. [Material UI Pagination](https://material-ui.com/components/pagination/)
