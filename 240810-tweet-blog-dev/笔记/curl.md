```
curl -X DELETE \
  http://127.0.0.1:3000/post/all \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk'


curl -X PUT \
  http://127.0.0.1:3000/post \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkU3RyIjoiYWRtaW4iLCJleHAiOjE3MjYyMTAyNDF9.pG_uYtwB4FXVTJcqQkdSJA-Tddi_OmO6QqvPzos-sZk' \
  -H 'Content-Type: application/json' \
  -d '{
  	"id": 6,
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
  http://127.0.0.1:3000/post/id/5 \
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