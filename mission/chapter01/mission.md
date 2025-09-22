

## 1. 리뷰 작성  ##

```basic
insert into review (user_id, store_id, content, star, created_at, updated_at)
values(user_id, store_id, ‘리뷰 내용’, 5, Now(), Now() )
```

→ 사진은 트랜잭션을 이용해서 삽입 가능할 듯 보임. 하나의 트랜잭션 안에 넣고 

일단 위 쿼리문을 사용해서 리뷰를 작성. 리뷰 아이디를 가져와서 review_photo에 사진을 삽입.


## 2. 마이페이지 화면 ##


```basic
select name, email, point, phone_number
from users
where users.id= my_user_id
```

내 user_id를 알고 있으므로 이를 기준으로 select문을 써서 가져오면 됨.

## 3. 진행중/ 진행완료 미션 ##

- 진행중 :  user-mission에서 is_completed가 false인 mission_id 를 통해서 

가게이름, 포인트, 데드라인, 최소금액 

```basic
select m.id , m.point, m.least_amount, m.store_id
from user-mission as um
join mission on mission.id = um.mission_id
join store on store.id = mission.store_id
where  m.is_completed = false
order by m.deadline asc, um.updated.at desc
limit 15;

```

- 진행완료 : user-mission에서 is_completed 가 true인 mission_id를 통해

 가게이름, 포인트, 최소금액

```basic
select m.store_id, m.point, m.least_amount
from user-mission as um
join mission on um.mission_id = mission.id
join store on mission.store_id = store.id
where m.is_completed = true
order by mission.created_at desc 
limit 15;
```

## 4. 홈 화면 쿼리 ##

```basic
select m.store_id, m.point, m.deadline, m.least_amount
from mission as m
join store as s on s.id = m.store_id
join location as l on l.id = s.location_id
where l.name = '안암동' AND m.id <10
order by m.deadline asc, m.point desc
limit 15;
```