# dvc_example
Getting started with Data Version Control and basic pipeline for NLP [check here](https://dvc.org/doc/get-started)

Main steps:
1. [Basic configure](https://dvc.org/doc/get-started/configure)
2. [Adding and storing data](https://dvc.org/doc/get-started/store-data)
3. [Connecting with code](https://dvc.org/doc/get-started/connect-code-and-data)
4. [Building pipeline](https://dvc.org/doc/get-started/pipeline)
5. [Creating metrics](https://dvc.org/doc/get-started/metrics)
6. [Making experiments](https://dvc.org/doc/get-started/compare-experiments)


# Dataset Registry
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
sudo chmod 774 shared_folder/
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
2. Создание и настройка data registry:
- Инициируем DVC проект: `dvc init`
- Добавляем в него нужные данные: `dvc add data/raw`
- Добавляем .dvc файлы в git репозиторий: `git add data/raw.dvc data/.gitignore`
- Коммитим изменения в проекте: `git commit -m 'added dvc for dataset versioning'`
- Добавляем нужный удалённый репозиторий для хранения файлов (в нашем случае - SSH к единому серверу):
```shell script
dvc remote add --local sshremote ssh://username@ip/home/path/to/project
dvc remote modify --local sshremote port 22
dvc remote modify --local sshremote user username
dvc remote modify --local sshremote keyfile path/to/keyfile
dvc remote modify --local sshremote ask_password true
```
- Отправляем добавленные данные в удалённый репозиторий: `dvc push -r sshremote`
3. Дальнейший workflow:
- В данном случае dataset registry - это именно удалённый git репозиторий
- Чтобы посмотреть текущее содержимое data registry: `dvc list -R link_to_git`
- dvc get | dvc import | dvc update
