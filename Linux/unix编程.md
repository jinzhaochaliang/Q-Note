#  unix编程

0. 一切皆文件

1. 目录操作

   1. ls
   2. cd
   3. pwd
   4. mkdir，rmdir 

2. 文件操作

   1. 命令
      1. cat，more，less，pg——查看文件内容
      2. cp——复制
      3. rm——删除
      4. mv——移动
      5. man——查看联机帮助
   2. who命令
      1. 通过读文件来获得需要的信息，每个登录的用户在文件中都有着对应的记录
   3. 函数
      1. open打开一个文件
      2. read从文件中读取数据
      3. close关闭文件
      4. create创建/重写文件
      5. write写文件

3. 应用缓冲技术对提高系统的效率是很明显的，它的主要思想是一次读入大量的数据放入缓冲区，需要的时候从缓冲区取得数据

4. 目录与文件属性

   1. 函数

      1. opendir

      2. readdir

      3. closedir

      4. seekdir，telldir，lseek

         ```c
         void do_ls(char []);
         
         int main(int ac,char * av[]){
             if(ac==1){
                 do_ls(".");
             }
             else{
                 while(--ac){
                     printf("%s:\n",*++av);
                     do_ls(*av);
                 }
             }
         }
         
         void do_ls(char dirname[]){
             DIR *dir_ptr;
             struct dirent *direntp;
             if((dir_ptr=opendir(dirname))==NULL){
                 fprintf(stderr,"ls1:cannot open %s\n",dirname);
             }
             else{
                 while((direntp=readdir(dir_ptr))!=NULL){
                     printf("%s\n",direntp->d_name);
                 }
                 closedir(dir_ptr);
             }
         }
         ```

5. 硬链接和软链接

   1. A是B的硬链接，则A的目录项中的节点号与B的目录项的节点号相同，即一个节点对应两个不同的文件名，两个文件名指向同一个文件。如果删除了其中一个，对另一个没有影响
   2. A是B的软链接，则A的目录项中的节点号与B的目录项的节点号不同，A与B指向两个不同的数据块，但是A中数据块存放的只是B的路径名，如果B删除了，A仍然存在，但是指向的是一个无效的链接

6. shell 进程控制和程序控制的一个工具

   1. 一个程序如何运行另一个程序

      1. 程序调用execvp

         ```c
         char *arglist[3];
         arglist[0] = "ls";
         arglist[1] = "-l";
         arglist[2] = 0; //注意结尾为0
         printf("About to exec ls -l\n");
         execvp(arglist[0],arglist);
         printf("ls is done.bye\n");
         ```

   2. 如何建立新的进程

      1. 一个进程调用fork（）来复制自己

      2. 进程调用fork，当控制转移到内核中的fork代码之后，内核分配新的内存块和内核数据结构，复制原来的进程到新的进程，向运行进程集添加新的进程，将控制返回给两个进程

      3. 建立一个新的进程

         ```c
         int ret_from_fork,mypid;
         mypid = getpid(); //jingchen ID
         printf("Before: my pid is %d\n",mypid);
         
         ret_from_fork= fork();
         sleep(1);
         printf("After: my pid is %d,fork() said %d\n",getpid(),ret_from_fork);
         ```

         新的进程从fork返回的地方开始执行，而不是从头开始执行

         所以打印一个before和两个After

      4. 子进程创建进程

         ```c
         printf("my pid is %d\n",getpid());
         fork();
         fork();
         fork();
         printf("my pid is %d\n",getpid());
         ```

         共打印八次

      5. 分辨子进程和父进程

         在子进程中fork（）返回为0

   3. 父进程如何等待子进程的退出

      1. 进程调用wait等待子进程退出

         ```c
         pid = wait(&status)
         ```

      2. 系统调用wait做两件事，首先wait暂停调用它的进程直到子进程结束。然后wait取得子进程结束时传给exit的值

      3. ```c
         int main(){
             int newpid;
             void child_code(),parent_code();
         
             printf("before:mypid is %d\n",getpid());
         
             if((newpid=fork())==-1)
                 perror("fork");
             else if(newpid==0)
                 child_code(DELAY);
             else
                 parent_code(newpid);
         }
         void child_code(int delay){
             printf("child %d here\n",getpid());
             sleep(DELAY);
             exit(17);
         }
         void parent_code(int childpid){
             int wait_rv;
             printf("test\n");
             wait_rv = wait(NULL);
             printf("done waiting for %d.wait returned: %d\n",childpid,wait_rv);
         }
         ```

         wait阻塞调用它的程序直到子进程结束，wait返回结束进程的PID

   4. shell如何运行程序

      1. 打印提示符write
      2. 取得命令read
      3. 建立新进程fork
      4. 等待子进程wait
         1. 子进程
            1. 运行新程序exec
            2. 新程序在运行main
            3. 新程序结束exit
      5. 得到子进程状态
      6. ......一直循环

