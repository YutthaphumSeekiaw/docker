build image

# command
docker build -t <ชื่อ image ที่ต้องการตั้ง> -f <ตำแหน่ง dockerfile> <path เริ่มต้นที่จะใช้ run docker file>

# example ที่เราจะ run
docker build -t node-server -f ./Dockerfile .
# ถ้าเกิดเป็น Dockerfile อยู่แล้ว (ไม่ใช่ชื่ออื่น) สามารถย่อเหลือเพียงแค่
docker build -t node-server .

ใช้คำสั่ง docker images ทำการ recheck image ดูว่า bulid เรียบร้อย

# To delete all the images,
docker rmi -f $(docker images -aq)

# command
docker run -p <port ภายนอก>:<port ภายใน> <ชื่อ image>

# example สำหรับ run เคสนี้
docker run -p 8000:8000 node-server

ทีนี้คำสั่ง docker run อาจจะไม่สะดวก เวลาที่อนาคตเราจะต้อง run container หลายตัว เนื่องจากตอนใช้คำสั่ง มันจะเป็นการ run แบบ foreground (เราจะอยู่หน้าจอ process นั้นไว้ด้วย) เราจะเปลี่ยนให้ docker run มา run แบบ background แทน

ทีนี้เราลอง ctrl+c ออกมาก่อน (ทำการ quit process) หากใคร quit ไม่ได้ ให้เปิด terminal (หรือ cmd ใหม่) แล้วใช้คำสั่งนี้

docker rm -f $(docker ps -a -q)

ถ้าถูกต้อง จะต้องยิง API localhost:8000 ไม่ได้แล้ว เปลี่ยนมาใช้คำสั่งนี้สำหรับการ run แทน

# เพิ่ม -d เพื่อให้สามารถ run background ได้
docker run -d -p 8000:8000 node-server

คำสั่ง docker ps ใช้สำหรับเช็ค container ที่กำลัง run อยู่ตอนนี้ได้

# command
docker logs <ชื่อ container>

# example (ดูตามชื่อข้างหลังใน docker ps)
docker logs node-server

###########################################################################################################################

รู้จักกับ docker-compose.yml ไฟล์
เราจะเริ่มต้นโดยการลองเปลี่ยน node เป็น docker-compose.yml กันก่อน โดย

เราจะใช้ Dockerfile อันเดิมที่เราสร้าง
เราจะเปลี่ยนจาก docker run เปลี่ยนเป็นการ run จาก docker-compose.yml
เราจะ start service ด้วย docker-compose up -d --build
ที่ docker-compose.yml เราจะมีหน้าตาแบบนี้

pattern ของ yaml file คือ key: value โดย ถ้าอันไหนเป็นลูกของ key ไหนก็จะเลื่อน tab ไป
ภายใต้ services = จำนวน container ที่จะ run ใน docker-compose

# Ex  docker-compose.yml

version: '3' # กำหนด docker version
services:
  node-server: # ตั้งชื่อ container (เหมือน --name)
    build: . # ตำแหน่ง dockerfile
    ports:
      - "8000:8000" # map port ออกมา เหมือน -p ใน docker run 

วางไว้ตำแหน่งเดียวกันกับ folder project แล้วลอง run ด้วยคำสั่ง

# คำสั่ง

docker-compose up เป็นคำสั่งสำหรับการ bulid และ run docker image ไปพร้อมๆกัน (docker-compose อำนวยความสะดวกในการ bulid + run ไปพร้อมกัน)
สามารถทำการ down service ทั้งหมดได้โดยใช้คำสั่ง docker-compose down ได้เช่นกัน
และกรณีที่ แค่ build แต่ไม่ run สามารถใช้เพียงแค่ docker-compose build


docker-compose up -d --build

docker-compose down 


#######################################################################################

Docker volume คือตัวที่ใช้สำหรับสร้าง persistent storage ที่ docker จะเก็บเอาไว้ (มองง่ายๆเป็นเหมือน disk จำลอง 1 ก้อน)

จะใช้สำหรับการเชื่อม data ที่อยู่ใน Container ออกมาภายนอก
โดยปกติ เคสของการ map data ใน Container ออกมาสู่ภายนอกจะมีอยู่ 2 เคส คือ
map ด้วยการ mount storage จริงในเครื่อ
map ด้วยการใช้ docker volume
ข้อดีของการใช้ docker volume คือ

มันจะสามารถสร้าง persistent storage ขึ้นมาได้ทันทีโดยไม่ต้องหา path ภายนอกเครื่อง (ทำให้ไม่รบกวน path ภายในเครื่อง)
รวมถึงตอนที่ลบ data สามารถใช้คำสั่ง docker volume ในการจัดการได้เลย (เดี๋ยวจะมีการแนะนำเพิ่มเติม)
คำถามคือ แล้วเคสไหนจำเป็นต้อง map data เก็บไว้ คำตอบก็คือ

เคสที่ต้องมีการเก็บข้อมูลจากฝั่ง user เช่น database ในที่นี่ก็คือ mysql (db) = เราจะเรียก container พวกนี้ว่าเป็นแบบ stateful คือ container ที่จะสามารถเปลี่ยนแปลงตาม data ที่เข้ามาได้

ถ้าเคสที่ไม่มี state ใดๆ (คือ user ไม่สามารถเปลี่ยนแปลงข้อมูลอะไรใน container ได้) = เราจะเรียก container พวกนี้ว่าเป็นแบบ stateless คือ container ที่ไม่สามารถเปลี่ยนแปลงตาม data ได้ (state จะเหมือนเดิมเสมอ ไม่เปลี่ยนแปลง)

# 1. map ด้วยการ mount path
services:
  db:
    image: mysql:latest
    container_name: db
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: tutorial
    ports:
      - "3306:3306"
    volumes:
      - /data:/var/lib/mysql

# 2. map ด้วย docker volume
services:
  ...
  db:
    image: mysql:latest
    container_name: db
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: tutorial
    ports:
      - "3306:3306"
    volumes:
      - mysql_data_test:/var/lib/mysql # เปลี่ยน path เป็น volume ที่เราสร้างไปด้านล่างใน docker-compose

volumes:
  mysql_data_test: # กำหนดชื่อ volume ที่ต้องการจะสร้าง
    driver: local



# วิธีการเข้าไปสำรวจใน container

# command
docker exec -it <container name> sh

# example
docker exec -it mysql sh

# ถ้าอันไหน support bash ก็ใช้ bash แทนได้
docker exec -it mysql bash


คำแนะนำอื่นๆเพิ่มเติม
เราสามารถนำ image ไป deploy ยังสถานที่ๆ สามารถนำ Image มา deploy เป็น Container ได้ เช่น

Google Cloud Registry (เก็บ Image) แล้วนำไป deploy บน Google Cloud run (Container)
Kubernetes ที่นำ image มา deploy เป็น Pod (Container)
แนะนำว่าศึกษาให้มากๆก่อนขึ้น deploy จริง ในประเด็นเรื่องของ

Network ที่ต้องดู เช่น port อะไรที่ควรออกข้างนอก, SSL certification, fireware
Monitoring กับ Logging
Scale อีกว่าจะทำยังไง
แนะนำว่าให้ศึกษาเรื่องพวกนี้ก่อนขึ้นจริง ไม่งั้น เจอปัญหา security กับ pricing แน่นอน
