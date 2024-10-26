### profile路由
```
curl -X PUT \
  http://127.0.0.1:3000/profile/external-links \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' \
  -H 'Content-Type: application/json' \
  -d '{
    "externalLinks": [
      {
        "uuid": "123e4567-e89b-12d3-a456-426614174000",
        "name": "GitHub",
        "description": "GitHub",
        "link": "https://github.com/username",
        "icon": "ff99d2c3-b1f7-430e-9d95-06c2959414a4",
        "isRadiu": true,
        "type": "contact"
      },
      {
        "uuid": "123e4567-e89b-12d3-a456-426614174001",
        "name": "LinkedIn",
        "description": "LinkedIn",
        "link": "https://linkedin.com/in/username",
        "icon": "ff99d2c3-b1f7-430e-9d95-06c2959414a4",
        "isRadiu": false,
        "type": "friend"
      }
    ]
  }'


curl -X DELETE \
  http://127.0.0.1:3000/profile/external-icon/not-used \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE'


curl -X DELETE \
  http://127.0.0.1:3000/profile/external-icon/uuid/ff99d2c3-b1f7-430e-9d95-06c2959414a4 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE'


curl -X PUT \
  http://127.0.0.1:3000/profile/avatar \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' \
  -H 'Content-Type: application/json' \
  -d '{
    "uuid": "89965fc3-3a78-4fa5-832b-cfbef91739ab"
  }'


curl -X DELETE \
  http://127.0.0.1:3000/profile/avatar/not-used \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE'


curl -X DELETE \
  http://127.0.0.1:3000/profile/avatar/uuid/90618a0d-0a13-41dd-9a16-c5d0cb0cf7f1 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE'


curl -X PUT \
  http://127.0.0.1:3000/profile/social-medias \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' \
  -H 'Content-Type: application/json' \
  -d '{
    "socialMedias": [
      {
        "name": "Twitter",
        "description": "Follow me on Twitter",
        "link": "https://twitter.com/username",
        "fontawesomeClass": "fa-brands fa-x-twitter"
      }
    ]
  }'


curl -X PUT \
  http://127.0.0.1:3000/profile/name-bio \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "haruki",
    "bio": "its MyGO"
  }'


curl -X GET \
  http://127.0.0.1:3000/profile/all \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' 


curl -X GET \
  http://127.0.0.1:3000/profile/store \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' 


curl -X GET \
  http://127.0.0.1:3000/profile/data \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' 


curl -X GET \
  http://127.0.0.1:3000/profile/test \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' 
  
```

### store测试
```
curl -X GET \
  http://127.0.0.1:3000/post/cursor/0 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' 

curl -X PUT \
  http://127.0.0.1:3000/image/config \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE' \
  -H 'Content-Type: application/json' \
  -d '{
    "imageLargeMaxLength": 1600,
  	"imageSmallMaxLength": 600,
  	"imageQuality": 85
}'

curl -X GET \
  http://127.0.0.1:3000/image/config \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzgzMTc4MDN9.xbor8AJI3qMROExyAn3tuKG2KCNxqaUYiKeQjPxyoNE'

curl -X POST \
  http://127.0.0.1:3000/public/admin-login \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "example_user",
    "password": "example_password"
}'

curl -X PUT \
  http://127.0.0.1:3000/admin/auth \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzIyNjY5MDF9.OGgabYEnDHvypgAlsVixc8l3Rz6kAfeeOkNnP1KuuOM' \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "example_user",
    "password": "example_password"
}'

curl -X GET \
  http://127.0.0.1:3000/admin/info \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MzIyNjY5MDF9.OGgabYEnDHvypgAlsVixc8l3Rz6kAfeeOkNnP1KuuOM' 
```

### 图片id查询 分页查询
```
curl -X GET \
  http://127.0.0.1:3000/image/cursor/0?havePost=false \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' 


curl -X GET \
  http://127.0.0.1:3000/image/cursor/0 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' 


curl -X GET \
  http://127.0.0.1:3000/image/id/1 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' 
```

### 帖子回收站查询
```
curl -X GET \
  http://127.0.0.1:3000/post/cursor/0?isDelete=true \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' 
```

### 不过滤isDelete的帖子id查询 
```
curl -X GET \
  http://127.0.0.1:3000/post/id/13?keepIsDetele=false \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'

curl -X GET \
  http://127.0.0.1:3000/post/id/13?keepIsDetele=true \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'

curl -X GET \
  http://127.0.0.1:3000/post/id/13 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'
```

### 测试：删除父帖
```
curl -X PUT \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
  	"id": 14,
    "isDeleted": true
}'

curl -X POST \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "replie post",
    "parentPostId": 14
}'

curl -X POST \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "parent post"
}'
```

### 其他
```
curl -X GET \
  http://127.0.0.1:3000/post/cursor/0 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' 
  

curl -X GET \
  http://127.0.0.1:3000/post/id/1 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'

curl -X POST \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "delete test",
    "images": [2],
    "parentPostId": 12
}'

curl -X PUT \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
  	"id": 13,
    "isDeleted": true
}'

curl -X POST \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "delete test",
    "images": [2],
    "parentPostId": 7
}'


curl -X GET \
  http://127.0.0.1:3000/public/post/id/1 

curl -X DELETE \
  http://127.0.0.1:3000/image/original/all \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'

curl -X DELETE \
  http://127.0.0.1:3000/image/original/id/2 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'


curl -X DELETE \
  http://127.0.0.1:3000/image/all \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'

curl -X DELETE \
  http://127.0.0.1:3000/image/id/4 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'

curl -X DELETE \
  http://127.0.0.1:3000/post/all \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'


curl -X PUT \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
  	"id": 9,
    "isDeleted": true 
}'


curl -X POST \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "",
    "images": [7,8]
}'


curl -X DELETE \
  http://127.0.0.1:3000/post/id/9 \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'


curl -X PUT \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
  	"id": 5,
    "isDeleted": true 
}'

curl -X PUT \
  http://127.0.0.1:3000/image/config \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "imageLargeMaxLength": 1800,
  	"imageSmallMaxLength": 600,
  	"imageQuality": 85
}'

curl -X GET \
  http://127.0.0.1:3000/image/config \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'

curl -X PATCH \
  http://127.0.0.1:3000/image \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "id": 8,
  	"alt": "Alternative Text"
}'


curl -X PUT \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
  	"id": 8,
    "content": "hello world!!!",
    "images": [5],
    "parentPostId": null
}'


curl -X POST \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "",
    "images": [1,2,3,4],
    "parentPostId": 1,
    "twitterId": "test"
}'


curl -X POST \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "",
    "images": [1,2,3,4],
    "parentPostId": 1
}'

curl -X POST \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "hello world",
    "images": [],
    "createdAt": "2024-04-04"
}'


curl -X PUT \
  http://127.0.0.1:3000/admin/info \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "jwtAdminExpSeconds": 2592000,
    "loginMaxFailCount": 10,
    "loginLockSeconds": 3600
}'


curl -X GET \
  http://127.0.0.1:3000/admin/info \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' 


curl -X PUT \
  http://127.0.0.1:3000/admin/auth \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "admin",
    "password": "adminadminadmin"
}'


curl -X POST \
  http://127.0.0.1:3000/public/admin-login \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "admin",
    "password": "adminadmin"
}'
```