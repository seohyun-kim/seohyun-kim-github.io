---
layout: single
title:  "CCTV_ver_1_app. 버튼 클릭 시 데이터 일부 변경(Ajax 사용하여 DB 접근 요청"
author: seohyun-kim
date: 2021-07-06 14:00
comments: true
---


[작동 영상 확인](https://www.youtube.com/watch?v=P29KqUuVSZo)

![image](https://user-images.githubusercontent.com/61939286/124620913-ce352100-deb4-11eb-945c-7898445bf864.png)

> 주기적으로 업데이트 되고 있는 데이터베이스의 최신값을 받아 표로 출력한다.  
버튼 클릭 시 서버에 데베접근을 요청하고 가장 최근값을 불러와 표의 데이터를 변경한다.  
  
  
# 초기에 GET 형식으로 DB의 가장 최근 값 가져옴
app.js
```js
app.get('/show', (req, res,next) => {
    conn.query(`SELECT * FROM test ORDER BY start_date DESC limit 1`, function(err,topics){
        if(err){
          console.log(err);
        }
        else{
          var data=topics;
          res.render(path.join(__dirname, './views/show.ejs'),
          {
            data: data,
          } );
        }
    });
});
```


# 테이블 형식에 가져온 최근 값 넣어줌
show.ejs
```html
<table>
  <script>

  var data = JSON.parse('<%- JSON.stringify(data) %>');

      document.write("<tr>");
        document.write("<th> Start Date </th>")
        document.write("<th> End Date </th>")
        document.write("<th> people </th>")
        document.write("<th> vehicle </th>")
       document.write("</tr>");

        document.write("<tr>");
          document.write("<td>" +	"<div class='s_time'>"+ data[0].start_date+"</div>" + "</td>");
          document.write("<td>" +	"<div class='e_time'>"+ data[0].end_date+"</div>" + "</td>");
          document.write("<td>" +	"<div class='people'>"+ data[0].people+"</div>" + "</td>");
          document.write("<td>" +	"<div class='vehicle'>"+ data[0].vehicle + "</div>" + "</td>");
        document.write("</tr>");

  </script>
</table>
 ```

# 버튼 클릭 시 서버에 요청 (Ajax)
show.ejs
```html
<button  id="refresh_btn" type="button"
				style="padding: 5px 20px; font-size:18px; ">
			Data Refresh</button>
```

```html
<script>
   $("#refresh_btn").on("click", function(){
                console.log("clicked");

                 $.ajax({
                     url : "/show",
                     type : "POST",
                     dataType : "JSON",
                 })

                 .done(function (json){
                      console.log(json);

                     $(".s_time").text(json.start_date)
                     $(".e_time").text(json.end_date)
                     $(".people").text(json.people)
                     $(".vehicle").text(json.vehicle)
                 })

                 .fail(function (xhr, status, errorThrown){
                     alert("Ajax failed")
                 })
             });

 </script>
```

# 서버에서 요청 처리 (DB 재 접근)
app.js
```js
app.post('/show', (req, res, next) => {
    conn.query(`SELECT * FROM test ORDER BY start_date DESC limit 1`, function(err,topics){
        if(err){
          console.log(err);
        }
        else{
            var responseData={};
            responseData=topics[0];
            console.log(responseData);
            res.json(responseData);
        }
    });
});

```

# json 형태로 받아온 값으로 데이터 일부 변경
show.ejs 에서 ajax 부분 일부
```html

 .done(function (json){
                      console.log(json);

                     $(".s_time").text(json.start_date)
                     $(".e_time").text(json.end_date)
                     $(".people").text(json.people)
                     $(".vehicle").text(json.vehicle)
                 })
```











