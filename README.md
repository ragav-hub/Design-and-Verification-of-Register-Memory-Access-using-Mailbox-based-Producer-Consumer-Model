# Experiment 5: Design and Verification of Register Memory Access using Mailbox-based Producer-Consumer Model

---

## Aim  
To design and verify a **Register Memory Access system** using a **mailbox-based Producer-Consumer model** in **SystemVerilog**, demonstrating inter-process communication and synchronization in a concurrent environment.

---

## Apparatus Required  
- Computer with **Windows/Linux OS**  
- **Synopsys VCS**  
- SystemVerilog source code editor  

---

## Description  
This experiment demonstrates **inter-process synchronization and data transfer** using the **mailbox mechanism** in SystemVerilog.  

- The **Producer process** writes data into a **mailbox**, simulating register write access.  
- The **Consumer process** retrieves data from the mailbox, representing register read access.  
- Mailboxes allow safe **communication between concurrent processes** without data corruption.  
- Verification ensures that the written data matches the read data (register consistency).  

---

## Features  
- Demonstrates **Producer–Consumer synchronization** using mailbox  
- Implements **Register Memory Access** (write and read)  
- Uses **concurrent processes** (`fork...join`) in SystemVerilog  
- Self-checking **testbench verification**  

---

## Procedure  

1. **Open Synopsys VCS**  
   - Launch the Synopsys VCS.  

2. **Create a New Project**  
   - `File → New → Project`  
   - Name it `Register_Memory`.  

3. **Add SystemVerilog Files**  
   - Create `register_memory.sv` → Register Memory design  
   - Create `register_memory_tb.sv` → Testbench with random input generator  

4. **Compile the Design and Testbench**  
   - Select both files, right-click → **Compile Selected**  
   - Resolve any syntax errors  

5. **Start Simulation**  
   - `VCS -full64 -sverilog Register_Memory.sv`  


6. **Add Signals to Waveform**  
   - Select clock, data, and output signals  

---

##  SystemVerilog Code

### Design File — `register_memory.sv`
```systemverilog
module register_memory #(parameter WIDTH = 8, DEPTH = 8) ();

    // Register memory
    logic [WIDTH-1:0] mem [0:DEPTH-1];

    // Write task
    task write_reg(input int addr, input logic [WIDTH-1:0] data);
        if (addr >= 0 && addr < DEPTH) begin
            mem[addr] = data;
            $display("[MEMORY] WRITE : Address = %0d, Data = %0h",
                     addr, data);
        end
        else begin
            $display("[MEMORY] ERROR : Invalid write address = %0d", addr);
        end
    endtask

    // Read task
    task read_reg(input int addr, output logic [WIDTH-1:0] data);
        if (addr >= 0 && addr < DEPTH) begin
            data = mem[addr];
            $display("[MEMORY] READ  : Address = %0d, Data = %0h",
                     addr, data);
        end
        else begin
            data = 'x;
            $display("[MEMORY] ERROR : Invalid read address = %0d", addr);
        end
    endtask

endmodule
```
### Testbench File
```
module register_memory_tb;

    parameter WIDTH = 8;
    parameter DEPTH = 8;

    // Mailbox for producer-consumer communication
    mailbox mbx = new();

    // Instantiate register memory
    register_memory #(WIDTH, DEPTH) regmem();

    // Transaction class
    class transaction;

        rand int addr;
        rand logic [WIDTH-1:0] data;
        bit is_write;

        constraint addr_range {
            addr inside {[0:DEPTH-1]};
        }

        function void display(string name);
            $display("[%s] Address = %0d, Data = %0h, Operation = %s",
                     name,
                     addr,
                     data,
                     is_write ? "WRITE" : "READ");
        endfunction

    endclass


    //=================================================
    // Producer
    //=================================================
    task producer();

        transaction tr;

        for (int i = 0; i < 10; i++) begin

            tr = new();

            assert(tr.randomize())
            else $fatal("Randomization failed");

            // Generate write transaction
            tr.is_write = 1;

            tr.display("PRODUCER");

            // Put transaction into mailbox
            mbx.put(tr);

            #5;
        end

        // End marker
        tr = new();
        tr.addr = 0;
        tr.data = 0;
        tr.is_write = 0;

        mbx.put(tr);

        $display("[PRODUCER] Finished producing transactions");

    endtask


    //=================================================
    // Consumer
    //=================================================
    task consumer();

        transaction tr;
        logic [WIDTH-1:0] read_data;

        forever begin

            // Get transaction from mailbox
            mbx.get(tr);

            // End condition
            if (!tr.is_write) begin
                $display("[CONSUMER] End of transactions");
                break;
            end

            tr.display("CONSUMER");

            // Perform memory write
            regmem.write_reg(tr.addr, tr.data);

            // Read back the value
            regmem.read_reg(tr.addr, read_data);

            // Verification
            if (read_data == tr.data) begin
                $display("[CHECK] PASS : Address = %0d, Expected = %0h, Read = %0h",
                         tr.addr, tr.data, read_data);
            end
            else begin
                $display("[CHECK] FAIL : Address = %0d, Expected = %0h, Read = %0h",
                         tr.addr, tr.data, read_data);
            end

            #2;
        end

        $display("[CONSUMER] Finished consuming transactions");

    endtask


    //=================================================
    // Main Test
    //=================================================
    initial begin

        $display("==============================================");
        $display(" REGISTER MEMORY MAILBOX VERIFICATION");
        $display("==============================================");

        fork
            producer();
            consumer();
        join

        #10;

        $display("==============================================");
        $display(" SIMULATION COMPLETED");
        $display("==============================================");

        $finish;

    end

endmodule
```
### Simulation Output

<img width="1919" height="1142" alt="Screenshot 2026-08-19 141940" src="https://github.com/user-attachments/assets/a3b27800-f018-4960-8d1b-66aacb06e665" />


----- 
<img width="1917" height="1127" alt="image" src="https://github.com/user-attachments/assets/584fc2ed-2f2f-413a-b6ab-b2d352c2d028" />

---
### Result

The Register Memory Access was successfully designed and verified using the mailbox-based Producer-Consumer model in SystemVerilog.
The producer wrote random data into memory, and the consumer read and validated it, demonstrating correct mailbox synchronization and data integrity.


