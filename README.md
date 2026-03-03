ex5 
#include <stdio.h>
#include <unistd.h>
#include <sys/shm.h>
#include <sys/ipc.h>
#include <sys/wait.h>

int main()
{
    int shmid;
    char *shmptr;

    shmid = shmget(2041, 32, 0666 | IPC_CREAT);

    if (fork() == 0) {
        // Child process — writing
        shmptr = shmat(shmid, NULL, 0);

        printf("Parent process writing:\n");

        for (int i = 0; i < 10; i++) {
            shmptr[i] = 'a' + i;
            putchar(shmptr[i]);
        }

        shmptr[10] = '\0';
        printf("\n");

        shmdt(shmptr);
    }
    else {
        wait(NULL);

        // Parent process — reading
        shmptr = shmat(shmid, NULL, 0);

        printf("Child process reading:\n");

        for (int i = 0; i < 10; i++)
            putchar(shmptr[i]);

        printf("\n");

        shmdt(shmptr);
        shmctl(shmid, IPC_RMID, NULL);
    }

    return 0;
}

ex 6
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <stdlib.h>

sem_t mutex;
int shared_resource = 0;

void *thread_function(void *arg)
{
    int i, val;
    long thread_id = (long)arg;

    for (i = 0; i < 5; i++) {
        sem_wait(&mutex);   // Lock the semaphore

        val = shared_resource;
        printf("Thread %ld: Read %d from shared resource\n", thread_id, val);

        val++;
        printf("Thread %ld: Writing %d to shared resource\n", thread_id, val);

        shared_resource = val;

        sem_post(&mutex);   // Unlock the semaphore
    }

    return NULL;
}

int main()
{
    pthread_t threads[3];
    int i, ret;

    // Initialize semaphore (1 = acts like mutex)
    sem_init(&mutex, 0, 1);

    // Create threads
    for (i = 0; i < 3; i++) {
        ret = pthread_create(&threads[i], NULL, thread_function, (void *)(long)i);
        if (ret != 0) {
            perror("pthread_create failed");
            return -1;
        }
    }

    // Wait for threads to finish
    for (i = 0; i < 3; i++) {
        ret = pthread_join(threads[i], NULL);
        if (ret != 0) {
            perror("pthread_join failed");
            return -1;
        }
    }

    printf("Final value of shared resource: %d\n", shared_resource);

    sem_destroy(&mutex);
    return 0;
}
