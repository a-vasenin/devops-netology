# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.  
Я не совсем понял вопрос. Имеется в виду builtin тип? Тогда, почему я так думаю: a) это базовая функциональность, б) `type cd` выводит такой тип  :)

2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.  
`grep -c <some_string> <some_file>`.

3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?  
`systemd`

4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?  
`ls 2> /dev/pts/1`

5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.  
`less < infile > outfile`

6. Получится ли вывести находясь в графическом режиме данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?  
`echo "Hello" > /dev/tty6`
Наблюдать выводимые данные в реальном времени не получится, но можно будет их увидеть, переключившись на tty из команды выше (CTRL + Alt + F6).

7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?  
`bash 5>$1` создает еще один процесс баша с дескриптором 5. Но я не понимаю, почему так происходит, почему вместо этого в текущем процессе не создается дескриптор 5. 
`echo netology > /proc/$$/fd/5` покажет в текущей сессии `netology`, потому что дескриптор 5 в ней перенаправлен в stdout.  
```
vagrant@vagrant:~$ ps -e | grep bash
   1116 pts/0    00:00:00 bash
vagrant@vagrant:~$ ls -la /proc/1116/fd
total 0
dr-x------ 2 vagrant vagrant  0 Jul 11 11:48 .
dr-xr-xr-x 9 vagrant vagrant  0 Jul 11 11:48 ..
lrwx------ 1 vagrant vagrant 64 Jul 11 11:48 0 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 11 11:48 1 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 11 11:48 2 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 11 11:49 255 -> /dev/pts/0

vagrant@vagrant:~$ bash 5>&1

vagrant@vagrant:~$ ps -e | grep bash
   1116 pts/0    00:00:00 bash
   1129 pts/0    00:00:00 bash
vagrant@vagrant:~$ ls -la /proc/1129/fd
total 0
dr-x------ 2 vagrant vagrant  0 Jul 11 11:50 .
dr-xr-xr-x 9 vagrant vagrant  0 Jul 11 11:50 ..
lrwx------ 1 vagrant vagrant 64 Jul 11 11:50 0 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 11 11:50 1 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 11 11:50 2 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 11 11:50 255 -> /dev/pts/0
lrwx------ 1 vagrant vagrant 64 Jul 11 11:50 5 -> /dev/pts/0

vagrant@vagrant:~$ echo netology > /proc/$$/fd/5
netology
```

8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.  
```
vagrant@vagrant:~$ cat no_file 5>&2 2>&1 1>&5 | grep file
cat: no_file: No such file or directory
```

9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?  
Выведет начальные переменные окружения для текущего процесса (bash).
Как аналогичный вывод получить, я не знаю. Потому что `env` и `printenv`, например, выводят текущие переменные, а не те, с которыми процесс запускался. Пруфцы: https://man7.org/linux/man-pages/man5/proc.5.html.

10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.  
В `/proc/<PID>/cmdline` хранится командная строка (команда + аргументы) процессы. Если процесс становится зомби, файл становится пустым.  
В `/proc/<PID>/exe` хранится символьная ссылка на путь до исполняемого файла.

11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.  
SSE4a
```
cat /proc/cpuinfo | grep sse
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm cmp_legacy cr8_legacy abm sse4a misalignsse 3dnowprefetch ssbd vmmcall fsgsbase avx2 rdseed clflushopt arat
```

12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

    ```bash
	vagrant@netology1:~$ ssh localhost 'tty'
	not a tty
    ```

	Почитайте, почему так происходит, и как изменить поведение.  

ssh не выделяет pty для удаленных команд. Чтобы его получить, нужно либо открыть ssh-сессию, а затем выполнять команды; либо использовать аргумент `-t`, который откроет pty:
```
vasenin@deb10:~/devops-netology/03-sysadmin-01-terminal$ ssh -t localhost 'tty'
vasenin@localhost's password: 
/dev/pts/2
Connection to localhost closed.
```

13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.  
Я не до конца понял, работает он или нет. В списке процессов, вроде, висит. ¯\_(ツ)_/¯
```
top
CTRL+Z
bg
disown top
screen
sudo reptyr -L $(pgrep top)
```

14. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.  
`tee` читает стандартный ввод и перенаправляет его на стандартный вывод и в файл. Разница с echo будет в том, что тут не наш шелл будет пытаться записать в рутовый файл, а tee, которая запущена с правами рута же.
