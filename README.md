
Сложность главный враг программиста.
Фред Брукс, различал два типа сложности:

* существенная сложность (essential complexity) - это сложность присущая самой решаемой задаче
* случайная сложность (acidental complexity) - сложность привнесенная реализацией, те та которой могло бы и не быть

Возможно в своей ежедневной деятельности вы испытывали подобные ощущения:

> решаешь типовую задачу, понимаешь ход решения, но реализация занимает значительно больше времени, чем казалось
бы нужно

> порой разбираешься с кодом, который должен решать конкретную задачу и даже понятно как, но реализация кажется сложнее, чем она должна быть, много лишних деталей и лишних сущностей

Когда мы говорим про красивую девушку - мы иногда говорим - "ничего лишнего". 

И действительно отсутствие лишнего (борьба со случайной сложностью) признак мастерства и хорошего вкуса у инженера. 

И мы хотим иметь инструменты соответсвующие нашему ...

Про clojure хочеться сказать - это язык в котором ничего лишнего. 
Это элегантный и мощный инструмент, который будет служить вам как швейцарский нож.

Итак clojure - это функциональный LISP исполняемый на JVM и написанный Rich Hickey.

В этом докладе я расскажу про clojure и почему этот язык хорошо подходит для современного web программирования.

Многие думают, что функциональное программирование это академическая область и там все сложно.

Но функциональная парадигма на самом деле очень проста:

Многие из вас хорошо представляют программирование с помощью процедур. Так вот получить из
процедурного программирования функциональное можно в два простых шага.

Необходимо лишь добавить понятия чистых функций и функций высшего порядка.

Чистые функции это функции, которые не имеют побочных эффектов и будучи вызванны с теми же аргументами
всегда будут возвращать одно и тоже значение. Как например сколько раз мы не брали бы синус от Pi/2 мы всегда будем получать единицу. По определению функции зависят только от параметров и являются идеальными компонентами системы, которые можно использовать в любом месте.

Функции высших порядков это функции, которые могут принимать или возвращать функции или иногда говорят - first-class functions. Те мы можем передавать и создавать вычесления также как мы передаем и модифицируем данные. Это мощный
механизм абстракции (например способный упразнить более половины паттернов банды четырех).

Я часто слышу - ФП хорош для математики и преобразования данных, но мы пишем web приложения, бизнес логику.

Давайте пристально посмотрим на то, как работает стандартное web приложение:

К нам приходит запрос в виде текстового сообщения (пока еще) в формате HTTP.
Мы разбираем этот текс и формируем объект Request.
Из объекта Request мы копируем данные в объекты. Если у вас правильная архитектура, то в дата-трансфер объекты,
из которых мы потом копируем данные в объекты модели предметной области. Потом наша ORM преобразуте это все в формат
понятный JDBC и в итоге данные оседают в бинарном формате в реляционной или не очень базе данных.
Что бы сформровать ответ все в обратном порядке до шаблонизатора, который отдает строчку HTML и ее мы заворачиваем
в HTTP ответ.

Но постойте львиную долю стэка составляет преобразование данных, для которого ФП хорош.

Продемонстрирую то, что большую часть нашего web стэка могут составлять чистые функции на примерах.

Шаблон должен получить данные и вернуть строчку или поток с HTML. Мы можем его выразить чистой функцией:

```clojure
(use 'hiccup.core)

(defn users-page [users]
  (html 
    [:html
      [:head ...]
      [:body
        [:ul
         (for [u users]
            [:li 
              [:a {:href (str "users/" (:id u)} (:name u)])]]])
```

Здесь мы исспользовали библиотеку `hiccup`, которая состоит в сущности из одной функции
`(html & content)`, принимающей на вход определенную структуру данных и преобразующая ее в HTML:

```
[:tag-name {:attr "value"} & content]
```

Итак одна функция и соглашение заменили нам templating движок. Причем композицию
наших шаблонов или как их иногда называют вьюх (view) и хэлперов (helper), 
мы можем реализовать как простую композицию функций.
Например сделать верхнеуровневый шаблон:

```clojure
(defn layout [& content]
   [:html
     [:head 
       [:style {:src ".."}]]
     [:body
       [:div.content content]]])
       
(defn users-page [users]
    (layout 
       [:ul (for [u users]
         [:li (:name u)]])]))
```

Другой пример это построение сложного SQL запроса с использованием,
например, honeysql - библиотеки реализующей такой же подход как мы уже видели.
В honeysql мы запрос представляем hash-mapом и абстракцию с композицией реализуем 
как функции и функциональная композиция:

```
(defn users []
  {:select [:*]  
   :from :users })

(defn active [q]
   (with-merge q [:= :users.active true]))
   
(defn with-role [q role]
   (with-join q 
      [[:user_roles :ur] [:= :ur.user_id :users.id]]
      [[:roles :r] [:= :ur.role_id :r.id] [:= :role.name role]]))

;; composition      
(-> (users)
    (admins)
    (with-role "admin"))
```




