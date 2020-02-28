# build image

`docker build --rm -t ibsps/jboss . `

# run this image

`docker volume create ibspdata`

`docker run -d --privileged --name app.local -v ibspdata:/ibspdata ibsps/jboss`

`docker run -d --privileged --name exe.local -v ibspdata:/ibspdata ibsps/jboss`

`docker run -d --privileged --name sch.local -v ibspdata:/ibspdata ibsps/jboss`