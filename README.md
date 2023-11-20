# Курс DevOps практичні завдання

## [Demo](Demo) - Реалізувати  вебсервіс у власному середовищі розробки 
Практичні завдання виконую ПК під Ubuntu 20.04.
Підготовка середовища:
1. Встановлюю minikube через Snap Store. Так само для зручності встановлюю k9s.  
2. В в VScode підключаю плагіни для Go, Python, Docker.
3. Реєструю і налаштовую доступ з робочого комп'ютера для сервісів [GitHub] (https://github.com/) i [Docker Hub] (https://hub.docker.com/)
4. Встановлюю компілятор Go:  

```bash
wget https://go.dev/dl/go1.21.4.linux-amd64.tar.gz
sudo -s
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.3.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version
```
`go version go1.21.4 linux/amd6`

5. Добре буде якщо зібрані та службові файли не будуть потрапляти на `github` отже створюю [.gitignore](.gitignore)

6. Збираю застосунок: 
```bash
cd demo 

go build -o bin/app src/main.go
bin/app

go run src/main.go
```

7. Роблю commit та push:  
```bash
git add *
git add .gitignore 
git commit -m "bin"
git push
```
Заходимо на github щоб пересвідчитись у відсутності там забороненої до синхронізації теки bin та її вмісту.

8. Запускаємо проект `bin/app` та перевіряємо його роботу через ```curl localhost:8080```  

9. Далі спробуємо запакувати розроблений вище код у контейнер та доставити його споживачу. Для цього нам потрібна буде реєстрація на [docker hub](https://hub.docker.com/) та встановлене розширення `Docker for Visual Studio Code` 

10. Створюємо в теці проекту `demo` файл [Dockerfile](./demo/Dockerfile)

11. Згенеруємо файл залежності для нашого проекту:

```bash
cd src
go mod init demo
cd -
```

12. Командою `docker build` ми запускаємо підготовку снапшоту нашої файлової системи за вказаною інструкцією що містить dockerfile:  

```bash
docker build .
```
Результатом буде записаний образ `writing image sha256:9cfa7d2b6bb336084b1f30bd58d18a0f67545d7ffa65b9df3ce04c8e3240ae7a` який включає наш бінарний сервер, контент та метадані. 

13. Запускаємо процес з контейнеру в ізольованому середовищі. Вказуємо додатково докеру порт на якому слухає наш сервер:  
`docker run -p 8080:8080 sha256:9cfa7d2b6bb336084b1f30bd58d18a0f67545d7ffa65b9df3ce04c8e3240ae7a`  
Перевіряємо роботу сервера за вказівками п.10, бачимо що працює все як раніше, але в ізольованому просторі. 

14. Зробимо наш продукт публічно доступним завантаживши його на докерхаб:
```bash
docker tag 9cfa7d2b6bb3 nazarcatkiller/app:v1.0.0
docker push nazarcatkiller/app:v1.0.0
```

15. Щоб набути трохи практичного досвіду спробуємо використати [образ автору курсу з репозиторію](https://hub.docker.com/r/denvasyliev/app/tags):  

```bash
# Забираємо образ на локальну машину:
docker pull denvasyliev/app:v1.0.1
# Запускаємо:
docker run -it --name den-container denvasyliev/app:v1.0.1
# Перевіряємо в іншому терміналі статус контейнеру:
docker ps
# Копіюємо контент з іміджу собі на локальний комп'ютер
docker cp 6bf76627fa44:/html ./DevOps/tmp
# Зупиняємо контейнер
docker stop den-container

``` 

16. Переходимо на етап деплойменту для чого створимо кубернетіс деплоймент з посиланням на наш докерхаб:

`kubectl create deploy demo --image nazarcatkiller/app:v1.0.0`
```
deployment.apps/demo created
```

17. Зробимо локальний порт-форвардінг в контейнер та перевіримо як це працює 

`kubectl port-forward deploy/demo 8080`
```
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
Handling connection for 8080
```
Отже сервер знову працює в ізольованому просторі за адресою `http://localhost:8080/` але вже на іншому рівні, тепер нам доступні інструменти управління, моніторингу та контролю Кубернетіс. 

18. Замикаємо процес розробки додаванням нових можливостей в програмний код нашого проектую. Для чого додамо в контент директорію svg файлів та змінимо код web-сторінки, щоб вона транслювала анімований сюжетний ряд. Після чого зберемо код та піднімемо версію проекту:

`docker build . -t nazarcatkiller/app:v1.0.1`  
`docker push nazarcatkiller/app:v1.0.1`

```
The push refers to repository [docker.io/nazarcatkiller/app]
4a002eb08023: Layer already exists 
b44788e15260: Pushed 
v1.0.1: digest: sha256:283124ac1ae4c43cfe7893af85feed2ebda570943440b3c9161a8d76388ec80f size: 738
```
19. Реалізуємо оновлення версії проекту на нову без переривання сервісу, для чого запускаємо в двох окремих терміналах наступні команди:
- Термінал 1 (команда оновлення версії):  
`kubectl get deploy demo -o wide`  
`kubectl set image deploy demo app=nazarcatkiller/app:v1.0.1`

- На другому терміналі вивід поточного стану на кластері  
`kubectl get po -w`
