Here is the **syntax** for calling each of the mentioned **MPI communication functions**:

---

### 1. **`MPI_Bcast`**  
Broadcasts a message from one process to all other processes in a communicator.  

```c
MPI_Bcast(void *buffer, int count, MPI_Datatype datatype, int root, MPI_Comm comm);
```
- `buffer`: Pointer to data being broadcast.  
- `count`: Number of elements in the buffer.  
- `datatype`: Type of data being broadcast (e.g., `MPI_INT`).  
- `root`: Rank of the broadcasting process.  
- `comm`: Communicator.

---

### 2. **`MPI_Scatter`**  
Distributes elements of an array from the root process to all processes in a communicator.

```c
MPI_Scatter(const void *sendbuf, int sendcount, MPI_Datatype sendtype, 
            void *recvbuf, int recvcount, MPI_Datatype recvtype, 
            int root, MPI_Comm comm);
```
- `sendbuf`: Pointer to the data to be sent (significant only at root).  
- `sendcount`: Number of elements sent to each process.  
- `sendtype`: Type of elements in the send buffer.  
- `recvbuf`: Pointer to the receive buffer.  
- `recvcount`: Number of elements received by each process.  
- `recvtype`: Type of elements in the receive buffer.  
- `root`: Rank of the sending process.  
- `comm`: Communicator.

---

### 3. **`MPI_Gather`**  
Gathers elements from all processes to the root process.

```c
MPI_Gather(const void *sendbuf, int sendcount, MPI_Datatype sendtype, 
           void *recvbuf, int recvcount, MPI_Datatype recvtype, 
           int root, MPI_Comm comm);
```
- `sendbuf`: Pointer to the data to send from each process.  
- `sendcount`: Number of elements sent by each process.  
- `sendtype`: Type of elements sent.  
- `recvbuf`: Pointer to the buffer where data is gathered (only significant at root).  
- `recvcount`: Number of elements received from each process.  
- `recvtype`: Type of elements received.  
- `root`: Rank of the receiving process.  
- `comm`: Communicator.

---

### 4. **`MPI_Reduce`**  
Performs a reduction operation (e.g., sum, max, etc.) on data from all processes and sends the result to the root process.

```c
MPI_Reduce(const void *sendbuf, void *recvbuf, int count, MPI_Datatype datatype, 
           MPI_Op op, int root, MPI_Comm comm);
```
- `sendbuf`: Pointer to the data to be reduced.  
- `recvbuf`: Pointer to the result buffer (only at root).  
- `count`: Number of elements to reduce.  
- `datatype`: Type of elements in the data.  
- `op`: Reduction operation (e.g., `MPI_SUM`, `MPI_MAX`).  
- `root`: Rank of the receiving process.  
- `comm`: Communicator.

---

### 5. **`MPI_Allreduce`**  
Performs a reduction operation on data from all processes and distributes the result to all processes.

```c
MPI_Allreduce(const void *sendbuf, void *recvbuf, int count, MPI_Datatype datatype, 
              MPI_Op op, MPI_Comm comm);
```
- `sendbuf`: Pointer to the data to be reduced.  
- `recvbuf`: Pointer to the result buffer.  
- `count`: Number of elements to reduce.  
- `datatype`: Type of elements in the data.  
- `op`: Reduction operation (e.g., `MPI_SUM`, `MPI_MAX`).  
- `comm`: Communicator.

---

### 6. **`MPI_Allgather`**  
Gathers data from all processes and distributes the entire result to all processes.

```c
MPI_Allgather(const void *sendbuf, int sendcount, MPI_Datatype sendtype, 
              void *recvbuf, int recvcount, MPI_Datatype recvtype, 
              MPI_Comm comm);
```
- `sendbuf`: Pointer to the data to send from each process.  
- `sendcount`: Number of elements sent by each process.  
- `sendtype`: Type of elements sent.  
- `recvbuf`: Pointer to the buffer to store the gathered data.  
- `recvcount`: Number of elements received from each process.  
- `recvtype`: Type of elements received.  
- `comm`: Communicator.

---

### 7. **`MPI_Alltoall`**  
Performs an **all-to-all personalized communication**, where each process sends distinct data to every other process.

```c
MPI_Alltoall(const void *sendbuf, int sendcount, MPI_Datatype sendtype, 
             void *recvbuf, int recvcount, MPI_Datatype recvtype, 
             MPI_Comm comm);
```
- `sendbuf`: Pointer to the data to send (distinct data for each process).  
- `sendcount`: Number of elements sent to each process.  
- `sendtype`: Type of elements sent.  
- `recvbuf`: Pointer to the buffer to store the received data.  
- `recvcount`: Number of elements received from each process.  
- `recvtype`: Type of elements received.  
- `comm`: Communicator.

---

### Summary Table:
| Function        | Purpose                                 | Data Flow            |
|-----------------|-----------------------------------------|----------------------|
| `MPI_Bcast`     | Broadcast data from root to all         | Root → All           |
| `MPI_Scatter`   | Distribute data from root to all        | Root → All           |
| `MPI_Gather`    | Gather data at the root                 | All → Root           |
| `MPI_Reduce`    | Reduce data at the root                 | All → Root           |
| `MPI_Allreduce` | Reduce data and distribute result       | All → All            |
| `MPI_Allgather` | Gather data and distribute to all       | All → All            |
| `MPI_Alltoall`  | Each process sends and receives data    | All ↔ All            |



Below are simple example scenarios and corresponding code for each **MPI function**:

---

### 1. **`MPI_Bcast`** - Broadcast data from root to all processes  
**Scenario**: Process 0 (root) has a value, and it broadcasts this value to all other processes.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size, data;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (rank == 0) {
        data = 100;  // Root initializes the data
        printf("Process 0 broadcasting data: %d\n", data);
    }

    MPI_Bcast(&data, 1, MPI_INT, 0, MPI_COMM_WORLD);

    printf("Process %d received data: %d\n", rank, data);

    MPI_Finalize();
    return 0;
}
```

---

### 2. **`MPI_Scatter`** - Scatter data from root to all processes  
**Scenario**: Process 0 has an array of data, and each process gets one part of it.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size;
    int sendbuf[8] = {1, 2, 3, 4, 5, 6, 7, 8}; // Array at root
    int recvbuf; // Each process receives one value

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    MPI_Scatter(sendbuf, 1, MPI_INT, &recvbuf, 1, MPI_INT, 0, MPI_COMM_WORLD);

    printf("Process %d received value: %d\n", rank, recvbuf);

    MPI_Finalize();
    return 0;
}
```

---

### 3. **`MPI_Gather`** - Gather data from all processes to the root  
**Scenario**: Each process sends its rank to the root process, which collects all values.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size;
    int sendbuf, recvbuf[8]; // Buffer to gather data

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    sendbuf = rank; // Each process sends its rank

    MPI_Gather(&sendbuf, 1, MPI_INT, recvbuf, 1, MPI_INT, 0, MPI_COMM_WORLD);

    if (rank == 0) {
        printf("Root process gathered data: ");
        for (int i = 0; i < size; i++) {
            printf("%d ", recvbuf[i]);
        }
        printf("\n");
    }

    MPI_Finalize();
    return 0;
}
```

---

### 4. **`MPI_Reduce`** - Perform a reduction operation at the root  
**Scenario**: Each process has a value, and the sum of all values is calculated at the root.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size;
    int sendbuf, recvbuf;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    sendbuf = rank + 1; // Example value: rank + 1

    MPI_Reduce(&sendbuf, &recvbuf, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);

    if (rank == 0) {
        printf("Sum of all values: %d\n", recvbuf);
    }

    MPI_Finalize();
    return 0;
}
```

---

### 5. **`MPI_Allreduce`** - Perform a reduction operation and distribute the result  
**Scenario**: Calculate the sum of all values and distribute the result to all processes.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size;
    int sendbuf, recvbuf;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    sendbuf = rank + 1; // Example value

    MPI_Allreduce(&sendbuf, &recvbuf, 1, MPI_INT, MPI_SUM, MPI_COMM_WORLD);

    printf("Process %d received sum: %d\n", rank, recvbuf);

    MPI_Finalize();
    return 0;
}
```

---

### 6. **`MPI_Allgather`** - Gather data from all processes and distribute it to all  
**Scenario**: Each process sends its rank, and all processes get the full list of ranks.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size;
    int sendbuf, recvbuf[8]; // Buffer to store gathered data

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    sendbuf = rank; // Send rank to all

    MPI_Allgather(&sendbuf, 1, MPI_INT, recvbuf, 1, MPI_INT, MPI_COMM_WORLD);

    printf("Process %d received data: ", rank);
    for (int i = 0; i < size; i++) {
        printf("%d ", recvbuf[i]);
    }
    printf("\n");

    MPI_Finalize();
    return 0;
}
```

---

### 7. **`MPI_Alltoall`** - All-to-all communication  
**Scenario**: Each process sends a unique value to every other process.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size;
    int sendbuf[4], recvbuf[4];

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Initialize send buffer with values to send to each process
    for (int i = 0; i < size; i++) {
        sendbuf[i] = rank * 10 + i;
    }

    MPI_Alltoall(sendbuf, 1, MPI_INT, recvbuf, 1, MPI_INT, MPI_COMM_WORLD);

    printf("Process %d received data: ", rank);
    for (int i = 0; i < size; i++) {
        printf("%d ", recvbuf[i]);
    }
    printf("\n");

    MPI_Finalize();
    return 0;
}
```

---

Each example includes simple **C code** that can be compiled with `mpicc` and executed using `mpirun`.
