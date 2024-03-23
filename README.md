#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_ALLOCATION 100

void* allocated_memory[MAX_ALLOCATION];
bool allocated_status[MAX_ALLOCATION] = {false};
int allocated_count = 0;

void* my_malloc(size_t size) {
    void* ptr = malloc(size);
    if (ptr != NULL) {
        allocated_memory[allocated_count] = ptr;
        allocated_status[allocated_count++] = true;
    }
    return ptr;
}

void my_free(void** ptr) {
    if (*ptr != NULL) {
        for (int i = 0; i < allocated_count; ++i) {
            if (allocated_memory[i] == *ptr) {
                if (allocated_status[i]) {
                    free(*ptr);
                    *ptr = NULL;
                    allocated_status[i] = false;
                } else {
                    fprintf(stderr, "Attempted to free unallocated memory\n");
                }
                return;
            }
        }
        fprintf(stderr, "Attempted to free unallocated memory\n");
    } else {
        fprintf(stderr, "Attempted to free unallocated memory\n");
    }
}

int main() {
    // Allocate memory
    int* mem1 = (int*)my_malloc(sizeof(int));
    int* mem2 = (int*)my_malloc(sizeof(int));
    int* mem3 = (int*)my_malloc(sizeof(int));
    
    my_free((void**)&mem1);
    // Check for not freeing allocated memory
    if(mem1 != NULL){
        printf("Memory leak detected: not freeing allocated memory\n");
    } else {
        printf("No memory leak detected: freeing allocated memory\n");
    }

    // Double free
    my_free((void**)&mem2);
    //my_free((void**)&mem2); // This will cause a double free error

    // Free mem3
    //my_free((void**)&mem3);

    // Accessing freed memory
    if (mem3 != NULL) {
        *mem3 = 10;
        printf("No memory leak detected: not accessing freed memory\n");
    } else {
        printf("Memory leak detected: accessing freed memory\n");
    }
    my_free((void**)&mem3);
    return 0;
}
