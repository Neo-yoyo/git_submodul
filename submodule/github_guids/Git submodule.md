# Git submodule

Подмодули позволяют сохранить один Git-репозиторий, как подкаталог другого Git-репозитория. При этом подмодуль остается отдельным репозиторием со своей собственной историей коммитов и состоянием. 

Подмодули используют, когда требуется сделать что-то с кодом подмодуля одновременно с работой над кодом основного проекта. 

### Добавление существующего Git-репозитория как подмодуля основного репозитория:

- Для добавления нового подмодуля используйте команду `git submodule add`, указав относительный или абсолютный URL проекта, который вы хотите начать отслеживать. В данном примере добавим библиотеку «`DbConnector`»:
  ```bash
  git submodule add https://github.com/chaconinc/DbConnector
  ```

  По умолчанию подмодуль сохраняется в каталог с именем репозитория, в примере выше — «`DbConnector`». Изменить каталог сохранения подмодуля можно указав путь к нему в конце команды.

- В репозиторий проеута будет добавлен новый файл `.gitmodules`. Это конфигурационный файл, в котором хранится соответствие между URL проекта и локальным подкаталогом, в который вы его выкачали: 
  ```bash
  [submodule "DbConnector"]
  	path = DbConnector
  	url = https://github.com/chaconinc/DbConnector
  ```

  Если подмодулей несколько - записей в файле будет несколько.

- Хоть `DbConnector` и является подкаталогом рабочего каталога, Git распознает его как подмодуль и не отслеживает его содержимое пока вы не перейдёте в него. Вместо этого, Git видит его как конкретный коммит этого репозитория. 
  Если вам нужен немного более понятный вывод, передайте команде `git diff` параметр `--submodule`. 

  ```bash
  git diff --cached --submodule
  ```

- Создание коммита:

  ```bash
  git commit -am 'Add DbConnector module'
  ```

  Права доступа `160000` - специальные права доступа в Git, которые означают, что вы сохраняете коммит как элемент каталога, а не как подкаталог или файл:

  ```bash
  $ git commit -am 'Add DbConnector module'
  [master fb9093c] Add DbConnector module
   2 files changed, 4 insertions(+)
   create mode 100644 .gitmodules
   create mode 160000 DbConnector
  ```

- Отправка изменений

  ```bash
  git push origin master
  ```



### Клонирование проекта с подмодулями

- Когда вы клонируете такой проект, по умолчанию вы получите каталоги, содержащие подмодули, но ни одного файла в них не будет.

- Вы должны выполнить две команды: 

  - `git submodule init` — для инициализации локального конфигурационного файла, и 
  - `git submodule update` — для получения всех данных этого проекта и извлечения соответствующего коммита, указанного в основном проекте.

  ```bash
  $ git submodule init
  Submodule 'DbConnector' (https://github.com/chaconinc/DbConnector) registered for path 'DbConnector'
  $ git submodule update
  Cloning into 'DbConnector'...
  remote: Counting objects: 11, done.
  remote: Compressing objects: 100% (10/10), done.
  remote: Total 11 (delta 0), reused 11 (delta 0)
  Unpacking objects: 100% (11/11), done.
  Checking connectivity... done.
  Submodule path 'DbConnector': checked out 'c3f01dc8862123d317dd46284b05b6892c7b29bc'
  ```

- Теперь каталог `DbConnector` находится в таком же состоянии, как и ранее при выполнении коммита.



Другой вариант сделать тоже самое:

- Если вы передадите опцию `--recurse-submodules` команде `git clone`, то она автоматически инициализирует и обновит каждый подмодуль в этом репозитории, включая вложенные подмодули (если подмодули репозитория имеют свои подмодули): 
  ```bash
  git clone --recurse-submodules https://github.com/chaconinc/MainProject
  ```

- Если вы уже клонировали проект, но не указали `--recurse-submodules`, вы можете объединить шаги `git submodule init` и `git submodule update`, запустив `git submodule update --init`. 

- Также для инициализации, получения и извлечения всех существующих вложенных подмодулей можно использовать безопасную команду `git submodule update --init --recursive`.



### Работа над проектом с подмодулями

- Если вы хотите проверить наличие изменений в подмодуле, перейдите в его каталог и выполните `git fetch`, а затем `git merge` для обновления локальной версии отслеживаемой ветки: 

  ```bash
  $ git fetch
  From https://github.com/chaconinc/DbConnector
     c3f01dc..d0354fc  master     -> origin/master
  $ git merge origin/master
  Updating c3f01dc..d0354fc
  Fast-forward
   scripts/connect.sh | 1 +
   src/db.c           | 1 +
   2 files changed, 2 insertions(+)
  ```

  Теперь если вы вернётесь в основной проект и выполните `git diff --submodule`, то увидите, что подмодуль обновился, а так же список новых коммитов. 

- Если сейчас вы создадите коммит, то состояние подмодуля будет зафиксировано с учётом последних изменений, что позволит другим людям их получить.

- Если вы не хотите вручную извлекать и сливать изменения в подкаталог, существует более простой способ сделать то же самое. Если вы выполните `git submodule update --remote`, то Git сам перейдёт в ваши подмодули, получит изменения и обновит их для вас:

  ```bash
  git submodule update --remote DbConnector
  ```

  Эта команда по умолчанию предполагает, что вы хотите обновить локальную копию до состояния ветки `master` из репозитория подмодуля. При желании вы можете задать другую ветку. Например, если вы хотите, чтобы подмодуль DbConnector отслеживал ветку `stable`, то можете указать это либо в файле `.gitmodules` (тогда и другие люди также будут отслеживать эту ветку), либо в вашем локальном файле `.git/config`:

  ```bash
  $ git config -f .gitmodules submodule.DbConnector.branch stable
  
  $ git submodule update --remote
  remote: Counting objects: 4, done.
  remote: Compressing objects: 100% (2/2), done.
  remote: Total 4 (delta 2), reused 4 (delta 2)
  Unpacking objects: 100% (4/4), done.
  From https://github.com/chaconinc/DbConnector
     27cf5d3..c87d55d  stable -> origin/stable
  Submodule path 'DbConnector': checked out 'c87d55d4c6d4b05ee34fbc8cb6f7bf4585ae6687'
  ```

  Если сейчас выполнить команду `git status`, то Git покажет нам наличие «новых коммитов» в подмодуле. 

- Если установить в настройках параметр `status.submodulesummary`, то Git будет также отображать краткое резюме об изменениях в ваших подмодулях: 

  ```bash
  $ git config status.submodulesummary 1
  
  $ git status
  ```



#### Получение изменений основного проекта из удалённого репозитория

- Простого запуска команды `git pull` для получения последних изменений недостаточно. По умолчанию, команда `git pull` рекурсивно получает изменения для подмодулей. При этом, она не **обновляет** подмодули. Для завершения обновления следует выполнить команду `git submodule update`:

  ```bash
  git submodule update --init --recursive
  ```

  Команду `git submodule update` лучше запускать с параметром `--init`, чтобы проинициализировать новые подмодули, которые, возможно, были добавлены в новых коммитах MainProject репозитория, а так же с параметром `--recursive`, чтобы обновить вложенные подмодули при их наличии.

- Если вы хотите автоматизировать этот процесс, то добавьте параметр `--recurse-submodules` при запуске команды `git pull`. Это приведёт к выполнению команды `git submodule update` сразу после получения и объединения изменений, приводя подмодули в корректное состояние. 





Внешние зависимости в гите: submodule или subtree?

https://mirrors.edge.kernel.org/pub/software/scm/git/docs/howto/using-merge-subtree.html

https://habr.com/ru/articles/419925/

https://habr.com/ru/articles/429014/

https://habr.com/ru/articles/428493/

https://www.atlassian.com/git/tutorials/git-subtree

git-subrepo и git-subtree
