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




Here are the **syntax**, **scenario**, and **code examples** for `MPI_Send`, `MPI_Recv`, and `MPI_Sendrecv`.

---

### 1. **`MPI_Send`** and **`MPI_Recv`**  
These are used for point-to-point communication between two processes.

#### **Syntax**:  
- `MPI_Send(buffer, count, datatype, dest, tag, comm)`  
- `MPI_Recv(buffer, count, datatype, source, tag, comm, status)`

- `buffer`: Data to send/receive.  
- `count`: Number of elements.  
- `datatype`: MPI data type (e.g., `MPI_INT`).  
- `dest/source`: Rank of destination/source process.  
- `tag`: Message identifier.  
- `comm`: Communicator (e.g., `MPI_COMM_WORLD`).  
- `status`: Information about the received message.

---

#### **Scenario**:  
Process 0 sends a message to Process 1, and Process 1 receives it.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size, data;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (size < 2) {
        printf("This program requires at least 2 processes.\n");
        MPI_Finalize();
        return 0;
    }

    if (rank == 0) {
        data = 42; // Data to send
        MPI_Send(&data, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
        printf("Process 0 sent data: %d to Process 1\n", data);
    } else if (rank == 1) {
        MPI_Recv(&data, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        printf("Process 1 received data: %d from Process 0\n", data);
    }

    MPI_Finalize();
    return 0;
}
```

---

### 2. **`MPI_Sendrecv`**  
This combines `MPI_Send` and `MPI_Recv` into a single operation. It is used for simultaneous sending and receiving of messages.

#### **Syntax**:  
- `MPI_Sendrecv(sendbuf, sendcount, sendtype, dest, sendtag, recvbuf, recvcount, recvtype, source, recvtag, comm, status)`

- `sendbuf`: Buffer for data to send.  
- `recvbuf`: Buffer for data to receive.  
- `sendcount/recvcount`: Number of elements.  
- `sendtype/recvtype`: Data types.  
- `dest/source`: Rank of destination/source process.  
- `sendtag/recvtag`: Message tags.  
- `status`: Status of the received message.

---

#### **Scenario**:  
Processes exchange data with each other. Process 0 sends data to Process 1 and receives data back from Process 1.

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int rank, size;
    int send_data, recv_data;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    if (size < 2) {
        printf("This program requires at least 2 processes.\n");
        MPI_Finalize();
        return 0;
    }

    if (rank == 0) {
        send_data = 100; // Data to send
        MPI_Sendrecv(&send_data, 1, MPI_INT, 1, 0, 
                     &recv_data, 1, MPI_INT, 0, 0, 
                     MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        printf("Process 0 sent %d and received %d\n", send_data, recv_data);
    } else if (rank == 1) {
        send_data = 200; // Data to send
        MPI_Sendrecv(&send_data, 1, MPI_INT, 0, 0, 
                     &recv_data, 1, MPI_INT, 1, 0, 
                     MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        printf("Process 1 sent %d and received %d\n", send_data, recv_data);
    }

    MPI_Finalize();
    return 0;
}
```

---

### **Summary**:

| **Function**     | **Purpose**                                                 |
|------------------|-------------------------------------------------------------|
| **`MPI_Send`**   | Sends a message to another process.                         |
| **`MPI_Recv`**   | Receives a message from another process.                    |
| **`MPI_Sendrecv`** | Combines sending and receiving data in one operation.     |


Each example demonstrates the syntax and functionality for `MPI_Send`, `MPI_Recv`, and `MPI_Sendrecv`.
