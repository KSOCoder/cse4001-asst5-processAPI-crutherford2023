# Assignment: Practicing the Process API
Practicing with fork, exec, wait. 

### Overview

In this assignment, you will practice using the Process API to create processes and run programs under Linux. The goal is to gain hands-on experience with system calls related to process management. Specifically, you will practice using the unix process API functions `fork()`, `exec()`, `wait()`, and `exit()`. 

⚠️ Note: This is not an OS/161 assignment. You will complete it directly on Linux. 

Use the Linux in your CSE4001 container. If you are using macOS, you may use the Terminal (you may need to install development tools with C/C++ compilers). 

**Reference Reading**: Arpaci-Dusseau, *Operating Systems: Three Easy Pieces*, Chapter 5 (Process API Basics)
 👉 [Chapter 5 PDF](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf)

---

### **Steps to Complete the Assignment**

1. **Set up your Repository**
   - Clone the assignment repository.
2. **Study the Reference Materials**
   - Read **Chapter 5**.
   - Download and explore the sample programs from the textbook repository:
      [OSTEP CPU API Code](https://github.com/remzi-arpacidusseau/ostep-code/tree/master/cpu-api).
3. **Write Your Programs**
   - Adapt the provided example code to answer the assignment questions.
   - Each program should be clear, well-commented, and compile/run correctly.
   - Add your solution source code to the repository.

4. **Prepare Your Report**
   - Answer the questions in the README.md file. You must edit the README.md file and not create another file with the answers. 
   - For each question:
     - Include your **code**.
     - Provide your **answer/explanation**.
5. **Submit Your Work via GitHub**
   - Push both your **program code** to your assignment repository.
   - This push will serve as your submission.
   - Make sure all files, answers, and screenshots are uploaded and rendered properly.








---
### Questions
1. Write a program that calls `fork()`. Before calling `fork()`, have the main process access a variable (e.g., x) and set its value to something (e.g., 100). What value is the variable in the child process? What happens to the variable when both the child and parent change the value of x?


```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// The value of variable x stays the same between the parent and child, while a new variable y changed in the child process changes to a new value.

int
main(int argc, char *argv[])
{
    int x = 100;
    int y = x;
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        y = 101;
        printf("Child: x = %d ; y = %d\n", (int) x, (int) y);
    } else {
        // parent goes down this path (original process)
        y = 102;
        printf("Parent: x = %d ; y = %d\n", (int) x, (int) y);
    }
    return 0;
}
```


2. Write a program that opens a file (with the `open()` system call) and then calls `fork()` to create a new process. Can both the child and parent access the file descriptor returned by `open()`? What happens when they are writing to the file concurrently, i.e., at the same time?

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <assert.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <sys/stat.h>

// Both the child and parent can open and write to the same file concurrently if it's accessed before the fork. If the file     is opened after the fork, they will open the same file, but different instances.

int
main(int argc, char *argv[])
{
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        int fd = open("./Q2.output", O_CREAT | O_WRONLY | O_TRUNC, S_IR>
        printf("Child has opened the file\n");
        if (fd < 0) {
           fprintf(stderr, "open failed\n");
           exit(1);
        }

        dup2(fd, STDOUT_FILENO);
        close(fd);

        char *myargs[3];
        myargs[0] = strdup("wc");
        myargs[1] = strdup("Q2.c");
        myargs[2] = NULL;
        execvp(myargs[0], myargs);
    } else {
        // parent goes down this path (original process)
        int wc = wait(NULL);
        printf("Parent has opened the file\n");
        assert(wc >= 0);
    }
    return 0;
}
  
```

3. Write another program using `fork()`.The child process should print “hello”; the parent process should print “goodbye”. You should try to ensure that the child process always prints first; can you do this without calling `wait()` in the parent?

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// Sleep works similiar to wait, but might not guarantee child first

int
main(int argc, char *argv[])
{
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        printf("Hello\n");
    } else {
        // parent goes down this path (original process)
        sleep(1);
        printf("Goodbye\n");
    }
    return 0;
}  
```


4. Write a program that calls `fork()` and then calls some form of `exec()` to run the program `/bin/ls`. See if you can try all of the variants of `exec()`, including (on Linux) `execl()`, `execle()`, `execlp()`, `execv()`, `execvp()`, and `execvpe()`. Why do you think there are so many variants of the same basic call?

```cpp
#define _GNU_SOURCE

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>

void run_variant(const char *label, void (*exec_func)(void)) {
   printf("\n=== %s ===\n", label);
   fflush(stdout);

   int rc = fork();
   if (rc < 0) {
      fprintf(stderr, "fork failed\n");
      exit(1);
   } else if (rc == 0) {
      exec_func();
      fprintf(stderr, "exec failed for %s\n", label);
      exit(1);
   } else {
      waitpid(rc, NULL, 0);
   }
}

// Pass full path + args individually to avoid path search
void variant_execl(void) {
   execl("/bin/ls", "ls", "-1", NULL);
}

// Same as execl but allows passing of a custom environment
void variant_execle(void) {
   char *myenv[] = { "MY_VAR=hello", NULL };
   execle("/bin/ls", "ls", "-1", NULL, myenv);
}

// Same as execl but searches PATH for program name
void variant_execlp(void) {
   execlp("ls", "ls", "-1", NULL);
}

// args pass as an array in vector form
void variant_execv(void) {
   char *myargs[] = { "ls", "-1", NULL };
   execv("/bin/ls", myargs);
}

// same as execv but includes PATH search
void variant_execvp(void) {
   char *myargs[] = { "ls", "-1", NULL };
   execvp("ls", myargs);
}

// same as execvp but includes custom environment support (and GNU exte>
void variant_execvpe(void) {
   char *myargs[] = { "ls", "-1", NULL };
   char *myenv[] = { "MY_VAR=hello", NULL };
   execvpe("ls", myargs, myenv);
}

int main(void) {
   run_variant("execl", variant_execl);
   run_variant("execle", variant_execle);
   run_variant("execlp", variant_execlp);
   run_variant("execv", variant_execv);
   run_variant("execvp", variant_execvp);
   run_variant("execvpe", variant_execvpe);

   return 0;
}
```

5. Now write a program that uses `wait()` to wait for the child process to finish in the parent. What does `wait()` return? What happens if you use `wait()` in the child?

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

// wait() returns the process ID of the child process that terminates. Therefore, unless a child has a child, wait() would return -1 as there are no children.

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        int rc_wait = wait(NULL);
        printf("child's wait returned: %d\n", rc_wait);
    } else {
        // parent goes down this path (original process)
        int rc_wait = wait(NULL);
        printf("hello, I am parent of %d (pid:%d)\n",
               rc, (int) getpid());
    }
    return 0;
} 
```

6. Write a slight modification of the previous program, this time using `waitpid()` instead of `wait()`. When would `waitpid()` be useful?

```cpp
// Add your code or answer here. You can also add screenshots showing your program's execution.  
```

7. Write a program that creates a child process, and then in the child closes standard output (`STDOUT FILENO`). What happens if the child calls `printf()` to print some output after closing the descriptor?

```cpp
// Add your code or answer here. You can also add screenshots showing your program's execution.  
```

