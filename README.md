# dvc_example
Getting started with Data Version Control and basic pipeline for NLP [check here](https://dvc.org/doc/get-started)

Main steps:
1. [Basic configure](https://dvc.org/doc/get-started/configure)
2. [Adding and storing data](https://dvc.org/doc/get-started/store-data)
3. [Connecting with code](https://dvc.org/doc/get-started/connect-code-and-data)
4. [Building pipeline](https://dvc.org/doc/get-started/pipeline)
5. [Creating metrics](https://dvc.org/doc/get-started/metrics)
6. [Making experiments](https://dvc.org/doc/get-started/compare-experiments)


## Shared Development Server
Задача:
- настройка хранения и версионирования всех данных сырых и промежуточных данных (вкл. модели)
- параллельный доступ 3-мя разработчиками к единым данным (без необходимости постоянного хранения их на локале)
- выстроение pipeline сбора и обработки данных
- работа через Jupyterhub
 
1. Настройка совместной работы через Jupyterhub:
- создание и настройка прав общей папки: 
```shell script
mkdir shared_folder/ 
sudo chown newowner:group shared_folder/
sudo chmod 775 shared_folder/
sudo chmod g+ws shared_folder/
```
- mount общей папки к отдельным папкам разработчиков: 
```shell script
sudo mount --bind shared_folder dev_folder
```
- mount работает в обе стороны! при внесении изменений одним разработчиком, всё меняется и для остальных 
- конфигурация настроек Jupyter Notebook:
```shell script
echo "import os" >> dev_folder/.jupyter/jupyter_notebook_config.py
echo "" >> dev_folder/.jupyter/jupyter_notebook_config.py
echo "os.umask(0o002)" >> dev_folder/.jupyter/jupyter_notebook_config.py 
```
- перезапуск сервера разработчика в Jupyterhub
- не забываем проверить, чтобы файлы из dev_folder/ были в .gitignore
2. Создание и настройка shared development server:
- Инициируем DVC проект: `dvc init  # at sshremote`
- Конфигурируем общий кэш:
```shell script
dvc config cache.dir /path/to/dvc-cache
dvc config cache.shared group
git add .dvc/config
git commit -m "dvc: shared external cache dir"
git push origin branch_name
```
- Добавляем локально нужный удалённый репозиторий для хранения файлов (в нашем случае - SSH к единому серверу):
```shell script
dvc remote add --local sshremote ssh://username@ip/home/path/to/project
dvc remote modify --local sshremote port 22
dvc remote modify --local sshremote user username
dvc remote modify --local sshremote keyfile path/to/keyfile
dvc remote modify --local sshremote ask_password true
```
- Добавляем нужные данные и .dvc файлы в git репозиторий: 
```shell script
git pull
dvc add data/raw
git add data/raw.dvc data/.gitignore
```
- Коммитим изменения в проекте: `git commit -m 'added dvc for dataset versioning' && git push`
- Отправляем добавленные данные в удалённый репозиторий: `dvc push -r sshremote`
3. Дальнейший workflow:
- В данном случае dataset registry - это именно удалённый git репозиторий
- Чтобы посмотреть текущее содержимое data registry: `dvc list -R link_to_git`
- Чтобы скачать реальный файл из удалённого репозитория, обязательно наличие .dvc файла локально (в месте инициации загрузки)
- Чтобы скачать файл: `dvc pull --remote sshremote` 
- Чтобы загрузить реальный файл: `dvc push --remote sshremote`
- Исходные данные храним в 1 папке (`raw`) и не вносим в них изменения
- Лучший подход к чистке данных с использованием DVC: 
    - 1 разработчик вносит какие-то изменения (производит очистку данных):
    ```shell script
        dvc add raw  # at sshremote
        dvc run -d raw -o clean ./cleanup_script.py raw clean  # at sshremote
        git add raw.dvc clean .dvc  # at sshremote
        git commit -m 'data processed with cleanup_script to clean'  # at sshremote
        git push origin branch_name  # at sshremote
    ```
    - 2 разработчик вносит свои изменения в данные:
    ```shell script
        git pull
        dvc checkout
        dvc run -d clean -o processed ./process.py clean process
        git add processed.dvc
        git commit -m "process clean data"
        git push origin branch_name
    ```
    - 3 разработчик легко подключается:
    ```shell script
      git pull
      dvc checkout
    ```
