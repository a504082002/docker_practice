## 指令
指令的一般格式為 `INSTRUCTION arguments`，指令包括 `FROM`、`MAINTAINER`、`RUN` 等。

### FROM
格式為 `FROM <image>`或`FROM <image>:<tag>`。

第一條指令必須為 `FROM` 指令。並且，如果在同一個Dockerfile中創建多個鏡像時，可以使用多個 `FROM` 指令（每個鏡像一次）。

### MAINTAINER
格式為 `MAINTAINER <name>`，指定維護者信息。

### RUN
格式為 `RUN <command>` 或 `RUN ["executable", "param1", "param2"]`。

前者將在 shell 終端中執行命令，即 `/bin/sh -c`；後者則使用 `exec` 執行。指定使用其它終端可以通過第二種方式實現，例如 `RUN ["/bin/bash", "-c", "echo hello"]`。

每條 `RUN` 指令將在當前鏡像基礎上執行指定命令，並提交為新的鏡像。當命令較長時可以使用 `\` 來換行。

### CMD
支持三種格式
* `CMD ["executable","param1","param2"]` 使用 `exec` 執行，推薦方式；
* `CMD command param1 param2` 在 `/bin/sh` 中執行，提供給需要交互的應用；
* `CMD ["param1","param2"]` 提供給 `ENTRYPOINT` 的默認參數；


指定啟動容器時執行的命令，每個 Dockerfile 只能有一條 `CMD` 命令。如果指定了多條命令，只有最後一條會被執行。

如果用戶啟動容器時候指定了執行的命令，則會覆蓋掉 `CMD` 指定的命令。

### EXPOSE
格式為 `EXPOSE <port> [<port>...]`。

告訴 Docker 服務端容器暴露的端口號，供互聯系統使用。在啟動容器時需要通過 -P，Docker 主機會自動分配一個端口轉發到指定的端口。

### ENV
格式為 `ENV <key> <value>`。
指定一個環境變量，會被後續 `RUN` 指令使用，並在容器執行時保持。

例如
```
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

### ADD
格式為 `ADD <src> <dest>`。

該命令將復制指定的 `<src>` 到容器中的 `<dest>`。
其中 `<src>` 可以是Dockerfile所在目錄的一個相對路徑；也可以是一個 URL；還可以是一個 tar 文件（自動解壓為目錄）。

### COPY
格式為 `COPY <src> <dest>`。

復制本地主機的 `<src>`（為 Dockerfile 所在目錄的相對路徑）到容器中的 `<dest>`。

當使用本地目錄為源目錄時，推薦使用 `COPY`。

### ENTRYPOINT
兩種格式：
* `ENTRYPOINT ["executable", "param1", "param2"]`
* `ENTRYPOINT command param1 param2`（shell中執行）。

配置容器啟動後執行的命令，並且不可被 `docker run` 提供的參數覆蓋。

每個 Dockerfile 中只能有一個 `ENTRYPOINT`，當指定多個時，只有最後一個起效。

### VOLUME
格式為 `VOLUME ["/data"]`。

創建一個可以從本地主機或其他容器掛載的掛載點，一般用來存放數據庫和需要保持的數據等。

### USER
格式為 `USER daemon`。

指定執行容器時的用戶名或 UID，後續的 `RUN` 也會使用指定用戶。

當服務不需要管理員權限時，可以通過該命令指定執行用戶。並且可以在之前創建所需要的用戶，例如：`RUN groupadd -r postgres && useradd -r -g postgres postgres`。要臨時獲取管理員權限可以使用 `gosu`，而不推薦 `sudo`。

### WORKDIR
格式為 `WORKDIR /path/to/workdir`。

為後續的 `RUN`、`CMD`、`ENTRYPOINT` 指令配置工作目錄。

可以使用多個 `WORKDIR` 指令，後續命令如果參數是相對路徑，則會基於之前命令指定的路徑。例如
```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
則最終路徑為 `/a/b/c`。

### ONBUILD
格式為 `ONBUILD [INSTRUCTION]`。

配置當所創建的鏡像作為其它新創建鏡像的基礎鏡像時，所執行的操作指令。

例如，Dockerfile 使用如下的內容創建了鏡像 `image-A`。
```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```

如果基於 image-A 創建新的鏡像時，新的Dockerfile中使用 `FROM image-A`指定基礎鏡像時，會自動執行 `ONBUILD` 指令內容，等價於在後面添加了兩條指令。
```
FROM image-A

#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
```

使用 `ONBUILD` 指令的鏡像，推薦在標簽中註明，例如 `ruby:1.9-onbuild`。

