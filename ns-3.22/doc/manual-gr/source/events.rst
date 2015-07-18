.. include:: replace.txt
.. highlight:: cpp


.. heading hierarchy:
   ------------- Chapter
   ************* Section (#.#)
   ============= Subsection (#.#.#)
   ############# Paragraph (no number)

.. Events and Simulator
--------------------

Γεγονότα και Προσομοιωτής
-------------------------


.. |ns3| is a discrete-event network simulator.  Conceptually, the simulator keeps track of a number of events that are scheduled to execute at a specified simulation time.  The job of the simulator is to execute the events in sequential time order. Once the completion of an event occurs, the simulator will move to the next event (or will exit if there are no more events in the event queue). If, for example, an event scheduled for simulation time "100 seconds" is executed, and the next event is not scheduled until "200 seconds", the simulator will immediately jump from 100 seconds to 200 seconds (of simulation time) to execute the next event. This is what is meant by "discrete-event" simulator.

Ο |ns3| έιναι ένα προσομοιωτής δικτύου διακριτών γεγονότων.  Εννοιολογικά, ο προσομοιωτής καταγράφει ένα πλήθος γεγονότων που έχουν προγραμματιστεί για να εκτελέστούν με μία προκαθορισμένη χρονική σειρά. Το αντικείμενο της εργασίας  του προσομοιωτή είναι να εκτελέσει τα γεγονότα, διαδοχικά (με διαδοχική χρονική σειρά). Με την ολοκλήρωση ενός γεγονότος, ο προσομοιωτής προχωρά στο επόμενο γεγονός (ή θα τερματίσει σε περίπτωση που δεν ευπάρχουν περισσότερα γεγονότα στην ουρά γεγονότων).Εάν, για παράδειγμα ένας γεγονός δρομολογείται να εκτελεστεί για την χρονική στιγμή προσομοίωσης "100 δευτερόλεπτα", και το επόμενο δεν έχει δρομολογηθεί πριν την χρονική στιγμή προσομοίωσης "200 δευτερόλεπτα", ο προσομοιωτής θα μεταπηδήσει ακαριαία από τα 100 δευτερόλεπτα στα 200 (του χρόνου προσομοιωσης) ώστε να εκτελέσει το επόμενο γεγονός. Αυτο εννοούμε με τον όρο προσωμοιωτής "διακριτών γεγονότων". 

.. To make this all happen, the simulator needs a few things:
Προκειμένου να πραγματοποιηθούν τα παραπάνω ο προσωμοιωτής χρειάζεται μερικά πράγματα:

.. 1) a simulator object that can access an event queue where events are 
   stored and that can manage the execution of events
.. 2) a scheduler responsible for inserting and removing events from the queue
.. 3) a way to represent simulation time
.. 4) the events themselves

1) ένα αντικείμενο προσωμοίωσης με προσβαση στην ουρά γεγονοτων, όπου βρίσκονται τα γεγονότα και το οποίο μπορεί να διαχειριστεί την εκτέλεση τους
2) έναν δρομολογητή υπεύθυνο  για την εκτέλεση των γεγονότων
3) έναν τρόπο να αναπαραστήσει τον χρόνο προσομοίωσης
4) τα ίδια τα γεγονότα

.. This chapter of the manual describes these fundamental objects (simulator, scheduler, time, event) and how they are used.

Η συγκεκριμένη ενότητα του εγχειριδίου χρήσης περιγράφει τα θεμελιώδη αντικείμενα (προσομοιωτής, δρομολογήτης, χρόνος, γεγονός) και πως χρησιμοποιούνται

.. Event
*****

Γεγονός
*******
.. *To be completed*

*Μη ολοκληρωμένο*

.. Simulator
*********

Προσομοιωτής
************

.. The Simulator class is the public entry point to access event scheduling facilities. Once a couple of events have been scheduled to start the simulation, the user can start to execute them by entering the simulator main loop (call ``Simulator::Run``). Once the main loop starts running, it will sequentially execute all scheduled events in order from oldest to most recent until there are either no more events left in the event queue or Simulator::Stop has been called.

Η κλάση Simulator (που αναπαριστά τον προσομοιωτή) αποτελεί ένα δημόσιο σημείο πρόσβασης προς δομές χρονοδομολόγησης γεγονότων. Μόλις χρονοδρομολογηθούν τα μερικά γεγονότα για την έναρξη της προσομοίωσης, ο χρήστης μπορεί να αρχίσει την προσομοίωση με την είσοδο στον κύριο βρόγχο επανάληψης (που ονομάζεται ``Simulator::Run``). Μόλις αρχίσει η εκτέλεση του κυρίως βρόγχου, θα εκτελεστούν σειριακά άλλα τα προγραμματισμένα γεγονότα, από την παλαιότερο προς το πιο προσφατο μέχρις ότου, είτε δεν υπάρχουν επιπλέον γεγονότα στην ουρά γεγονότων,  είτε έχει πραγματοποιηθεί κλήση της μεθόδου Simulator::Stop.   

.. To schedule events for execution by the simulator main loop, the Simulator class provides the Simulator::Schedule* family of functions.

Για τον χρονοπρογραμματισμό των γεγονότων προς εκτέλεση από τον κύριο βρόγχο, η κλάση Simulator παρέχει την οικογένεια μεθόδων Simulator::Schedule*

.. 1) Handling event handlers with different signatures

1) Χειρισμός διαχειριστών γεγονότων με διαφορετικές υπογραφές

.. These functions are declared and implemented as C++ templates to handle automatically the wide variety of C++ event handler signatures used in the wild. For example, to schedule an event to execute 10 seconds in the future, and invoke a C++ method or function with specific arguments, you might write this:

Οι συγκεκριμένες συναρτήσεις δηλώνονται και υλοποιούνται σαν C++ πρότυπα (C++ templates) για τoν αυτόματo χείρισμο μία ποικιλίας C++ διαχειριστών γεγονότων που χρησιμοποιούνται χωρίς έλεγχο. Για παράδειγμα, για τον χρονοπρογραμματισμό για την εκτλέση 10 δευτερολέπτων στο μέλλον, και την κλήση μία C++ μεθόδου ή συνάρτησης με συγκεκριμένα ορίσματα, μπορείτε να χρησιμοποιήσετε τον παρακάτω κώδικα: 

::

   void handler (int arg0, int arg1)
   {
     std::cout << "handler called with argument arg0=" << arg0 << " and
        arg1=" << arg1 << std::endl;
   }

   Simulator::Schedule(Seconds(10), &handler, 10, 5);

.. Which will output:
Το οποίο θα έχει ως αποτέλεσμα:

.. sourcecode:: text

  handler called with argument arg0=10 and arg1=5


.. Of course, these C++ templates can also handle transparently member methods on C++ objects:

προφανώς, τα συγκεκριμένα πρότυπα C++ θα μπορούν να χειριστούν με διαφάνεια, μεθόδους μέλη σε αντικείμενα C++:

.. *To be completed:  member method example*

*Μη ολοκληρωμένο: παράδειγμα μεθόδου μελους*

.. Notes:

Σημειώσεις:

.. * the ns-3 Schedule methods recognize automatically functions and methods only if they take less than 5 arguments. If you need them to support more arguments, please, file a bug report.

* Οι μέθοδοι χρονοδρομολόγησης του ns-3  αναγνωρίζουν αυτόματα συναρτήσεις και μεθόδους μόνο εάν χρησιμοποιούν λιγότερα από 5 ορίσματα. Εάν χρειάζεστε υποστήριξη για παραπάνω από 5 ορίσματα, παρακαλούμε να υποβάλετε αντίστοιχη αναφορά σφαλμάτων (bug report)

.. * Readers familiar with the term 'fully-bound functors' will recognize the Simulator::Schedule methods as a way to automatically construct such objects. 

* Αναγνώστες που γνωρίζουν τον όρο 'fully-bound functors' θα αναγνωρίσουν τις μεθόδους Simulator::Schedule ως ένα τρόπο για την αυτόματη κατασκευή τέτοιου είδους αντικειμένων. 

.. 2) Common scheduling operations

2) Απλές λειτουργίες χρονοπρογραμματισμού 

.. The Simulator API was designed to make it really simple to schedule most events. It provides three variants to do so (ordered from most commonly used to least commonly used):

Η διεπαφή επικοινωνίας του Προσομοιωτή (Simulator API) έχει σχεδιαστεί ώστε να είναι πολύ εύκολη διαδικασία η χρονοδρομολόγηση των περίσσότερων γεγονότων. Παρέχει τρεις εναλλακτικές λύσεις για να επιτευχθεί το ζητούμενο οι οποίες (ταξινομημένες από την πιο συχνά χρησιμοποιούμενη έως την λιγότερο συχνά χρησιμοποιούμενη) είναι:

.. * Schedule methods which allow you to schedule an event in the future by providing the delay between the current simulation time and the expiration date of the target event.
* Οι μέθοδοι χρονοδρομολόγησης που επιτρέπουν την χρονοδομολόγηση ενός γεγονότος στο μέλλον,  παρέχοντας την καθυστέρηση μεταξύ του τρέχοντος χρόνου προσομοίωσης και της ημερομήνίς λήξης του γεγονότος στόχου. 
  
.. * ScheduleNow methods which allow you to schedule an event for the current simulation time: they will execute _after_ the current event is finished executing but _before_ the simulation time is changed for the next event.
* Οι μέθοδοι ScheduleNow που επιτρέπουν την χρονοδορμολόγηση ενός γεγονότος για τον τρέχοντα χρόνο προσομοίωσης: θα εκτελέσουν _μετά_ την ολοκλήρωση του γεγονότος που εκτελείται αύτη την στιγμή αλλά _πριν_ ο χρόνος προσομοίωσης αλλάξει για το επόμενο γεγονός.

.. * ScheduleDestroy methods which allow you to hook in the shutdown process of the Simulator to cleanup simulation resources: every
  'destroy' event is executed when the user calls the Simulator::Destroy method.
* Οι μέθοδοι ScheduleDestroy που επιτρέπουν την προσδεση (hook) στην διαδικαία τερματισμού του Προσομοιωτή (shutdown process) για τον καθαρισμό των πόρων της προσομοίωσης: κάθε γεγονός 'destroy' εκτελειται όταν ο χρήστης καλέσει την μέθοδο Simulator::Destroy


.. 3) Maintaining the simulation context

3) Διαχείρηση του περιβάλλοντος (συμφραζομένων) προσομοίωσης


.. There are two basic ways to schedule events, with and without *context*.
Υπάρχουν δύο κυρίως τρόποι για την χρονοδρομολόγηση γεγονότων, με ή χωρίς συμφραζόμενα.
.. What does this mean?
Τι σημαίνει αυτό;

::

  Simulator::Schedule (Time const &time, MEM mem_ptr, OBJ obj);

vs.

::

  Simulator::ScheduleWithContext (uint32_t context, Time const &time, MEM mem_ptr, OBJ obj);

.. Readers who invest time and effort in developing or using a non-trivial simulation model will know the value of the ns-3 logging framework to debug simple and complex simulations alike. One of the important features that is provided by this logging framework is the automatic display of the network node id associated with the 'currently' running event. 



The node id of the currently executing network node is in fact tracked
by the Simulator class. It can be accessed with the
Simulator::GetContext method which returns the 'context' (a 32-bit
integer) associated and stored in the currently-executing event. In some
rare cases, when an event is not associated with a specific network
node, its 'context' is set to 0xffffffff.

To associate a context to each event, the Schedule, and ScheduleNow
methods automatically reuse the context of the currently-executing event
as the context of the event scheduled for execution later. 

In some cases, most notably when simulating the transmission of a packet
from a node to another, this behavior is undesirable since the expected
context of the reception event is that of the receiving node, not the
sending node. To avoid this problem, the Simulator class provides a
specific schedule method: ScheduleWithContext which allows one to
provide explicitly the node id of the receiving node associated with
the receive event.

*XXX: code example*

In some very rare cases, developers might need to modify or understand
how the context (node id) of the first event is set to that of its
associated node. This is accomplished by the NodeList class: whenever a
new node is created, the NodeList class uses ScheduleWithContext to
schedule a 'initialize' event for this node. The 'initialize' event thus executes
with a context set to that of the node id and can use the normal variety
of Schedule methods. It invokes the Node::Initialize method which propagates
the 'initialize' event by calling the DoInitialize method for each object
associated with the node. The DoInitialize method overridden in some of these
objects (most notably in the Application base class) will schedule some
events (most notably Application::StartApplication) which will in turn
scheduling traffic generation events which will in turn schedule
network-level events.

Notes:

* Users need to be careful to propagate DoInitialize methods across objects
  by calling Initialize explicitely on their member objects
* The context id associated with each ScheduleWithContext method has
  other uses beyond logging: it is used by an experimental branch of ns-3
  to perform parallel simulation on multicore systems using
  multithreading.

The Simulator::* functions do not know what the context is: they
merely make sure that whatever context you specify with
ScheduleWithContext is available when the corresponding event executes
with ::GetContext.

It is up to the models implemented on top of Simulator::* to interpret
the context value. In ns-3, the network models interpret the context
as the node id of the node which generated an event. This is why it is
important to call ScheduleWithContext in ns3::Channel subclasses
because we are generating an event from node i to node j and we want
to make sure that the event which will run on node j has the right
context.

.. Time
****

Χρόνος
******

.. *To be completed*

*Μη ολοκληρωμένο*

.. Scheduler
*********

Δρομολογητής
************

..*To be completed*

*Μη ολοκληρωμένο*
