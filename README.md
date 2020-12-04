ΑΡΧΙΤΕΚΤΟΝΙΚΗ ΥΠΟΛΟΓΙΣΤΩΝ


Χαϊτίδης Δημήτριος 9310
Email: chaitidi@ece.auth.gr
Πέτσα Δήμητρα 9102
Email: petsadimitr@ece.auth.gr

Εργαστήριο Β / Ομάδα 5

Μέρος πρώτο


Πρώτο ερώτημα: 

Απο την εντολή που χρησιμοποιήσαμε στο terminal για την καταστκευή του simulation παρατηρούμε τον τύπο του επεξεργαστή:

./build/ARM/gem5.opt -d hello_resultconfigs/example/arm/starter_se.py--cpu="minor" "tests/test-progs/hello/bin/arm/linux/hello"

cpu_type - Minor

Απο την συνάρτηση main του αρχείου starter_se.py μπορούμε να παρατηρήσουμε τα παρακάτω χαρακτηριστικά:

cpu_frequency - 4Ghz
parser.add_argument("--cpu-freq", type=str, default="4GHz")
 
number_of_cores - 1
parser.add_argument("--num-cores", type=int, default=1, help="Number of CPU cores")
 
memory_type - DR3_1600_8x8
parser.add_argument("--mem-type", default="DDR3_1600_8x8", choices=ObjectList.mem_list.get_names(), help = "type of memory to use")
 
memory_channels - 2
parser.add_argument("--mem-channels", type=int, default=2, help = "number of memory channels")
memory_size - 2GB
parser.add_argument("--mem-size", action="store", type=str, default="2GB", help="Specify the physical memory size")
 
Απο την συνάρτηση init της κλάσης SimpleSeSystem γίνεται αντιληπτό ότι το σύστημα εχει τα εξής:
Δύο cache lines με το καθε ένα να έχει μέγεθος γραμμής 64 bytes
if self.cpu_cluster.memoryMode() == "timing": self.cpu_cluster.addL1() self.cpu_cluster.addL2(self.cpu_cluster.clk_domain)
cache_line_size = 64
Voltage_Domain - 3.3V
self.voltage_domain = VoltageDomain(voltage="3.3V")
Clock_Domain - 1Ghz
self.clk_domain = SrcClockDomain(clock="1GHz", voltage_domain=self.voltage_domain)
 
Δεύτερο ερώτημα :
a)
Στο αρχείο config.ini μπορούμε να δούμε και να επαληθεύσουμε τα παρακάτω:
Cache_size & Memory_size & Memory_channels
11 [system]
15 cache_line_size=64
21 mem_ranges=0:2147483648
22 memories=system.mem_ctrls0.dram system.mem_ctrls1.dram
 
 
 
Clock_Domain :
42 [system.clk_domain]
43 type=SrcClockDomain
44 clock=1000
 
Cpu_frequency :
56 [system.cpu_cluster.clk_domain]
57 type=SrcClockDomain
58 clock=250 #250ns => F = 4GHz
Cpu_type :
64 [system.cpu_cluster.cpus]
65 type=MinorCPU
Voltage_Domain :
1649 [system.voltage_domain]
1650 type=VoltageDomain
1651 eventq_index=0
1652 voltage=3.3
 
 
b) 
Ο αριθμός των commited instructions είναι ίσος με 5028.
 system.cpu_cluster.cpus.committedInsts 5028 # Number of instructions committed
Τα στατιστικά της gem5 δείχνουν 5834.
system.cpu_cluster.cpus.committedOps 5834 # Number of ops commited
Αυτό οφείλεται στο γεγονός ότι οι "committedInsts" είναι ο αριθμός των assembly εντολών που εκτελέστηκαν, ενώ οι "committedOps" ο αριθμός των micro-operations. Κάθε εντολή μπορεί να επεκταθεί σε ένα μεγαλυτέρο αριθμό micro-operations, οπότε οι "committedInsts" είναι πάντα μεγαλύτερες ή ίσες με των "committedOps". Ουσιαστικά τα micro-ops είναι ένας τρόπος για να μετατρέψεις τις CISC-εντολές σε RISC-εντολές. Αυτό έχει ως αποτέλεσμα να απλοποιούνται οι περισσότερο περιπλόκες εντολές που δεν μπορούσαν να προσπελαστούν γρήγορα από έναν επεξεργαστή, να ολοκληρώνονται σε συντομότερο χρονικό διαστημα και να επιτυγχάνεται υψηλότερη ταχύτητα ρολογιού.
 
c)
Ο αριθμός των προσπελάσων της L2 cache βρίσκεται στο stats.txt και ισούται με 479.
system.cpu_cluster.l2.demand_accesses::total 479 # number of demand (read+write) accesses
Στο L2 memory system που χρησιμοποιείται σε αυτή την περίπτωση, γίνεται πρόσπελαση στην μνήμη l2 cache όταν έχουμε miss στο L1 memory system δηλαδή στην L1 dcache και L1 icache. Από το stats.txt βλέπουμε ότι έχουμε 179 misses στην dcache και 332 στην icache.
system.cpu_cluster.cpus.dcache.overall_misses::total 179 # number of overall misses
system.cpu_cluster.cpus.icache.overall_misses::total 332 # number of overall misses
 
Προσθέτοντας το σύνολο των misses στις δυο cache βλέπουμε ότι από τα συνολικά misses που είχαμε στο πρώτο επίπεδο, 32 από αυτά δεν μεταβιβάστηκαν στην l2. Αυτό συνέβη επειδή τα συγκεκριμένα misses εντοπίστηκαν και διαχειρίστηκαν στους MSHR καταχωρητές και δεν χρειάστηκε να γίνει κατανομή στην l2.
system.cpu_cluster.cpus.dcache.overall_mshr_hits::total 32 # number of overall MSHR hits




Review
H εργασία δεν ήταν ιδιαίτερα πολύπλοκη στην υλοποίηση της. Τα προβληματα που συναντήσαμε ήταν κυρίως στην εγκατάσταση του gem5.Τα ερωτήματα που αφορούσαν το θεωρητικό κομμάτι του μαθήματος οδήγησαν να καταλάβουμε καλύτερα ή να μάθουμε καινούργιες έννοιες της αρχιτεκτονικής υπολογιστών, ψάχνοντας το documentation του gem5 αλλά και πηγές του διαδικτύου. Πέρα από τα θέματα που αφορούν το γνωστικό αντικείμενο του μαθήματος, ασχοληθήκαμε με το περιβάλλον των ubuntu και το github, με αποτέλεσμα να εξοικοιωθούμε περισσότερο με αυτά, κάτι που ήταν ιδιαίτερα θετικό. Επιπλέον ήταν αρκετά ενδιαφέρον το πως λειτουργήσαμε συνολικά στο περιβάλον.
