 #  EXPERIMENT 7 CloudSim Multi-Datacenter Simulation

## Overview
This project demonstrates a **CloudSim 3.0.3 simulation** where two users submit cloudlets that are executed across **two separate datacenters**. Each datacenter contains one host, and tasks are scheduled through a broker that distributes workloads to virtual machines.

This experiment illustrates **multi-datacenter architecture, user-level task scheduling, and distributed cloud execution behavior**.

---

## Aim
To create **two datacenters with one host each** and execute cloudlets from **two users** to analyze distributed execution behavior.

---

## Objectives
- Simulate multiple datacenters in CloudSim
- Configure hosts and allocate resources
- Simulate multiple users submitting tasks
- Create virtual machines for execution
- Analyze distributed execution results

---

## Tools and Technologies

| Tool | Version |
|------|---------|
| Java JDK | 1.8+ |
| CloudSim | 3.0.3 |
| IDE | IntelliJ / Eclipse |
| OS | Windows / Linux |

---

## System Configuration

### Datacenter Configuration

| Parameter | Value |
|----------|------|
| Datacenters | 2 |
| Hosts per DC | 1 |
| RAM | 2048 MB |
| Storage | 1,000,000 MB |
| Bandwidth | 10,000 Mbps |
| Scheduler | Time Shared |

---

### Virtual Machine Configuration

| Parameter | VM1 | VM2 |
|----------|-----|-----|
| MIPS | 500 | 500 |
| RAM | 512 MB | 512 MB |
| BW | 1000 Mbps | 1000 Mbps |
| VMM | Xen | Xen |
| Scheduler | Time Shared | Time Shared |

---

### Cloudlet Configuration

| Parameter | Value |
|----------|------|
| Users | 2 |
| Cloudlets | 4 |
| Lengths | 20000, 40000, 60000, 80000 |
| File Size | 300 MB |
| Output Size | 300 MB |
| PEs | 1 |

---

## Algorithm
1. Initialize CloudSim library  
2. Create two datacenters  
3. Configure one host per datacenter  
4. Create broker  
5. Create virtual machines  
6. Submit VM list to broker  
7. Create cloudlets with different lengths  
8. Submit cloudlets  
9. Start simulation  
10. Collect results  

---

 ## Program flow 

 <img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/fdb0238c-1ae5-41ff-9648-321aac189b36" />


## Program

import org.cloudbus.cloudsim.Cloudlet;
import org.cloudbus.cloudsim.CloudletSchedulerTimeShared;
import org.cloudbus.cloudsim.Datacenter;
import org.cloudbus.cloudsim.DatacenterBroker;
import org.cloudbus.cloudsim.DatacenterCharacteristics;
import org.cloudbus.cloudsim.Host;
import org.cloudbus.cloudsim.Pe;
import org.cloudbus.cloudsim.Storage;
import org.cloudbus.cloudsim.UtilizationModel;
import org.cloudbus.cloudsim.UtilizationModelFull;
import org.cloudbus.cloudsim.Vm;
import org.cloudbus.cloudsim.VmAllocationPolicySimple;
import org.cloudbus.cloudsim.VmSchedulerTimeShared;

import org.cloudbus.cloudsim.core.CloudSim;

import org.cloudbus.cloudsim.provisioners.BwProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.PeProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.RamProvisionerSimple;

import java.util.*;

public class ScalableSimulation {

    private static List<Vm> vmList;
    private static List<Cloudlet> cloudletList;

    public static void main(String[] args) {

        try {

            int numUsers = 1;
            Calendar calendar = Calendar.getInstance();
            boolean traceFlag = false;

            CloudSim.init(numUsers, calendar, traceFlag);

            // ---------- SCALABLE PARAMETERS ----------
            int numDatacenters = 2;
            int hostsPerDatacenter = 2;
            int numberOfVMs = 2;
            int numberOfCloudlets = 10;

            // ---------- CREATE DATACENTERS ----------
            for (int i = 0; i < numDatacenters; i++) {
                createDatacenter("Datacenter_" + i, hostsPerDatacenter);
            }

            // ---------- CREATE BROKER ----------
            DatacenterBroker broker = createBroker();
            int brokerId = broker.getId();

            // ---------- CREATE VMs ----------
            vmList = new ArrayList<>();

            for (int i = 0; i < numberOfVMs; i++) {
                int mips = 1000 + (i * 200);
                long size = 10000;
                int ram = 1024;
                long bw = 1000;
                int pesNumber = 1;
                String vmm = "Xen";

                Vm vm = new Vm(i, brokerId, mips, pesNumber, ram, bw, size,
                        vmm, new CloudletSchedulerTimeShared());

                vmList.add(vm);
            }

            broker.submitVmList(vmList);

            // ---------- CREATE CLOUDLETS ----------
            cloudletList = new ArrayList<>();
            UtilizationModel utilizationModel = new UtilizationModelFull();

            for (int i = 0; i < numberOfCloudlets; i++) {

                long length = 40000 + (i * 5000);
                long fileSize = 300;
                long outputSize = 300;
                int pesNumber = 1;

                Cloudlet cloudlet = new Cloudlet(i, length, pesNumber,
                        fileSize, outputSize,
                        utilizationModel,
                        utilizationModel,
                        utilizationModel);

                cloudlet.setUserId(brokerId);
                cloudletList.add(cloudlet);
            }

            broker.submitCloudletList(cloudletList);

            // ---------- START SIMULATION ----------
            CloudSim.startSimulation();

            List<Cloudlet> newList = broker.getCloudletReceivedList();

            CloudSim.stopSimulation();

            printCloudletList(newList);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // -----------------------------------------------------
    // CREATE DATACENTER
    // -----------------------------------------------------
    private static Datacenter createDatacenter(String name, int hostsCount) {

        List<Host> hostList = new ArrayList<>();

        for (int i = 0; i < hostsCount; i++) {

            List<Pe> peList = new ArrayList<>();

            int mips = 1000;

            for (int j = 0; j < 4; j++) {
                peList.add(new Pe(j, new PeProvisionerSimple(mips)));
            }

            int hostId = i;
            int ram = 8192;
            long storage = 1000000;
            int bw = 10000;

            hostList.add(
                    new Host(
                            hostId,
                            new RamProvisionerSimple(ram),
                            new BwProvisionerSimple(bw),
                            storage,
                            peList,
                            new VmSchedulerTimeShared(peList)
                    )
            );
        }

        String arch = "x86";
        String os = "Linux";
        String vmm = "Xen";
        double timeZone = 10.0;
        double costPerSec = 3.0;
        double costPerMem = 0.05;
        double costPerStorage = 0.001;
        double costPerBw = 0.0;

        DatacenterCharacteristics characteristics =
                new DatacenterCharacteristics(
                        arch, os, vmm, hostList,
                        timeZone, costPerSec,
                        costPerMem, costPerStorage,
                        costPerBw);

        Datacenter datacenter = null;
        try {
            datacenter = new Datacenter(name, characteristics,
                    new VmAllocationPolicySimple(hostList),
                    new LinkedList<Storage>(), 0);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return datacenter;
    }

    // -----------------------------------------------------
    // CREATE BROKER
    // -----------------------------------------------------
    private static DatacenterBroker createBroker() {

        DatacenterBroker broker = null;
        try {
            broker = new DatacenterBroker("Broker");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return broker;
    }

    // -----------------------------------------------------
    // PRINT RESULTS
    // -----------------------------------------------------
    private static void printCloudletList(List<Cloudlet> list) {

        System.out.println("\n========== OUTPUT ==========");
        System.out.println("Cloudlet ID\tStatus\tVM ID\tTime\tStart\tFinish");

        for (Cloudlet cloudlet : list) {

            System.out.print(cloudlet.getCloudletId() + "\t\t");

            if (cloudlet.getStatus() == Cloudlet.SUCCESS) {
                System.out.print("SUCCESS\t");
                System.out.println(
                        cloudlet.getVmId() + "\t" +
                                cloudlet.getActualCPUTime() + "\t" +
                                cloudlet.getExecStartTime() + "\t" +
                                cloudlet.getFinishTime()
                );
            }
        }
    }
}

 ## Output 

 <img width="653" height="341" alt="Screenshot 2026-03-09 101024" src="https://github.com/user-attachments/assets/97a1aa90-ecf9-45e5-acb1-87163947181f" />





## Result

The simulation successfully executed tasks submitted by multiple users across two datacenters. Cloudlets were scheduled based on available resources and virtual machine allocation and executed accordingly.
