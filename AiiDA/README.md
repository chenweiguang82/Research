# AiiDA
## 安装

按照[官方安装教程](https://aiida-core.readthedocs.io/en/latest/install/installation.html#installation)来操作，一般会正常安装上去。
*这是我用的平台*
- ArchLinux
```shell
(base) [zznu@archlinux ~]$ cat /proc/version 
Linux version 5.1.14-arch1-1-ARCH (builduser@heftig-10038) (gcc version 9.1.0 (GCC)) #1 SMP PREEMPT Sat Jun 22 16:28:48 UTC 2019
```
- Miniconda3

```shell
conda install virtualenv
virtualenv ~/.virtualenvs/aiida
source .virtualenvs/aiida/bin/activate
pip install --pre aiida-core
```
不过我安装的时候最后出了点小问题
```shell
gcc -pthread -shared -B /home/zznu/miniconda3/compiler_compat -L/home/zznu/miniconda3/lib -Wl,-rpath=/home/zznu/miniconda3/lib -Wl,--no-as-needed -Wl,--sysroot=/ build/temp.linux-x86_64-3.7/psutil/_psutil_common.o build/temp.linux-x86_64-3.7/psutil/_psutil_posix.o build/temp.linux-x86_64-3.7/psutil/_psutil_linux.o -o build/lib.linux-x86_64-3.7/psutil/_psutil_linux.cpython-37m-x86_64-linux-gnu.so
/home/zznu/miniconda3/compiler_compat/ld: build/temp.linux-x86_64-3.7/psutil/_psutil_common.o: unable to initialize decompress status for section .debug_info
/home/zznu/miniconda3/compiler_compat/ld: build/temp.linux-x86_64-3.7/psutil/_psutil_common.o: unable to initialize decompress status for section .debug_info
/home/zznu/miniconda3/compiler_compat/ld: build/temp.linux-x86_64-3.7/psutil/_psutil_common.o: unable to initialize decompress status for section .debug_info
/home/zznu/miniconda3/compiler_compat/ld: build/temp.linux-x86_64-3.7/psutil/_psutil_common.o: unable to initialize decompress status for section .debug_info
build/temp.linux-x86_64-3.7/psutil/_psutil_common.o: file not recognized: file format not recognized
collect2: error: ld returned 1 exit status
error: command 'gcc' failed with exit status 1
----------------------------------------
ERROR: Command "/home/zznu/.virtualenvs/aiida/bin/python -u -c 'import setuptools, tokenize;__file__='"'"'/tmp/pip-install-gjo0q7jx/psutil/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /tmp/pip-record-hq33lo_q/install-record.txt --single-version-externally-managed --compile --install-headers /home/zznu/.virtualenvs/aiida/include/site/python3.7/psutil" failed with error code 1 in /tmp/pip-install-gjo0q7jx/psutil/ 
```
搜索了下，这个错误应该是最新的binutils包的bug，退回到*binutils-2.30-5-x86_64*就可以解决。
`sudo pacman -U http://archive.virtapi.org/packages/b/binutils/binutils-2.30-5-x86_64.pkg.tar.xz`
## 配置
### 数据库安装
Aiida使用PostgreSQL来存储信息，安装：
`sudo pacman -S postgresql`
### 数据库配置
- 初始化数据库
```
sudo su - postgres -c "initdb --locale en_US.UTF-8 -E UTF8 -D '/var/lib/postgres/data'"
```
- 切换到postgres
`su - postgres`
- 启动程序
`psql`
- 创建一个账户，用于AiiDA
```
CREATE USER aiida WITH PASSWORD 'aiida';
```
- 创建数据库
```
CREATE DATABASE aiidadb OWNER aiida ENCODING 'UTF8' LC_COLLATE='en_US.UTF-8' LC_CTYPE='en_US.UTF-8' TEMPLATE=template0;
```
- 授权
```
GRANT ALL PRIVILEGES ON DATABASE aiidadb to aiida;
```
- 退出
```
\q
```
- 测试
```
psql -h localhost -d aiidadb -U aiida -W
```
输入密码，成功的话会出现提示符
### 初始化Aiida
- 切换到普通账户
```
su - zznu
```
- 使用virtualenv
```
source ~/.virtualenvs/aiida/bin/activate
```
- 设置Aiida
```
verdi setup
```
##这是我用的一个测试配置##
```
Default user email: chenweiguang82@126.com
Database engine: postgresql_psycopg2
PostgreSQL host: localhost
PostgreSQL port: 5432
AiiDA Database name: aiidadb
AiiDA Database user: aiida
AiiDA Database password: <password>
AiiDA repository directory: /home/zznu/.aiida/repository/test
[...]
Configuring a new user with email 'chenweiguang82@126.com'
First name: Weiguang
Last name: Chen
Institution: Zhengzhou Normal University
The user has no password, do you want to set one? [y/N] y
Insert the new password:
Insert the new password (again):
```
- 启动Aiida守护进程
```
verdi daemon start
```
输出为：
```
Starting the daemon... RUNNING
```
- 查看守护进程状态
```
verdi daemon status
```
输出为：
```
Profile: test
Daemon is running as PID 1826 since 2019-07-22 08:13:20
Active workers [1]:
  PID    MEM %    CPU %  started
-----  -------  -------  -------------------
 1830    0.118        0  2019-07-22 08:13:20
Use verdi daemon [incr | decr] [num] to increase / decrease the amount of workers
```
- 查看verdi状态
```
verdi status
```
输出为：
```
 _ profile:     On profile test
 _ repository:  /home/zznu/.aiida/repository/test
 _ postgres:    Connected as aiida@localhost:5432
 _ rabbitmq:    Connected to amqp://127.0.0.1?heartbeat=600
 _ daemon:      Daemon is running as PID 1826 since 2019-07-22 08:13:20
 ```
 这里要说明一下，上面的rabbitmq需要安装，不安装的话会执行上面命令会出现错误。
 - 安装rabbitmq
 ```
 pacman -S rabbitmq
 systemctl start rabbitmq.service
 systemctl enable rabbitmq.service
 ```
 - 安装支持的作业调度器  
 一直以来我都用torque，现在想试试slurm
 查了下AUR里有slurm，不过需要munge，因此先安装了munge
 ```pacman -S yaourt
 yaourt -S munge
 ```
 接下来安装slurm
