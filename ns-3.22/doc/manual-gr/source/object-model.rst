.. include:: replace.txt
.. highlight:: cpp

.. _Object-model:

.. Object model

Το μοντέλο αντικειμένων
-----------------------

.. |ns3| is fundamentally a C++ object system. Objects can be declared and instantiated as usual, per C++ rules. |ns3| also adds some features to traditional C++ objects, as described below, to provide greater functionality and features. This manual chapter is intended to introduce the reader to the |ns3| object model.

Ο |ns-3| είναι ουσιαστικά ένα σύστημα αντικειμένων C++. Τα αντικείμενα μπορολυν να δηλωθούν και να αρχικοποιηθούν ως συνήθως, σύμφωνα με τους κανόνες της C++. O |ns-3| προσθέτει μετικά επιπλέον χαρακτηριστικά στα παραδοσιακά C++ αντικείμενα, όπως θα περιγράψουμε στην συνέχεια, ώστε να παρέχει μεγαλυτερη λειτουργικότητα και χαρακτηριστικά. Η συγκεκριμένη ενότητα του εγχειριδίου έχει ως σκοπό να ειδάγει τον αναγώστη στο μοντέλο αντικειμένων του |ns-3|.

.. This section describes the C++ class design for |ns3| objects. In brief, several design patterns in use include classic object-oriented design (polymorphic interfaces and implementations), separation of interface and implementation, the non-virtual public interface design pattern, an object aggregation facility, and reference counting for memory management. Those familiar with component models such as COM or Bonobo will recognize elements of the design in the |ns3| object aggregation model, although the |ns3| design is not strictly in accordance with either.

Η συγκεκριμένη ενότητα περιγράφει τον σχεδιασμό κλάσεων C++ για τα αντικείμενα του |ns3|. Εν συντομία, διάφορα σχεδιαστικά πρότυπα που χρησιμοποιούνται περιλαμβάνουν κλασσικά αντικειμοστραφή σχέδια (πολυμορφικές διεπαφές και υλοποιήσεις), διαχωρισμό της διεπαφής επικοινωνίας και της υλοποίησης, σχεδιαστικά πρότυπα για μη-εικονικές δημόσιες διεπαφές, μία εγκατάσταση συνάθροισης αντικειμένων (object aggregation facility), και καταμέτρηση αναφορών για την διαχείριση μνήμης. Όσοι είναι εξοικειωμένοι με μοντέλα συστατικών (component) όπως το COM ή το Bonobo θα αναγνωρίσουν στοιχεία από τον σχεδιασμό τους στο συναθροιστικό μοντέλο του |ns3|, παρόλο που ο σχεδιασμός του |ns3| δεν συμφωνεί αυστηρά με κανένα από τα δύο μοντέλα.  

.. Object-oriented behavior

Αντικειμενοστραφής συμπεριφορά
******************************

.. C++ objects, in general, provide common object-oriented capabilities (abstraction, encapsulation, inheritance, and polymorphism) that are part of classic object-oriented design. |ns3| objects make use of these properties; for instance::

Τα C++ αντικείμενα, γενικά, περιέχουν κοινές αντικειμενοστραφείς ικανότητες (αφαίρεση, ενθυλάκωση, κληρονομηκότητα και πολυμορφισμό) που αποτελούν μέρος του κλασσικού αντικειμεστραφούς μοντέλου. Τα αντικείμενα |ns3| κάνουν χρήση των συγκεκριμένων ιδιοτήτων. Για παράδειγμα::

    class Address
    {
    public:
      Address ();
      Address (uint8_t type, const uint8_t *buffer, uint8_t len);
      Address (const Address & address);
      Address &operator = (const Address &address);
      ...
    private:
      uint8_t m_type;
      uint8_t m_len;
      ...
    };

.. Object base classes

.. Οι βασικές κλάσεις Object
****************************

.. There are three special base classes used in |ns3|. Classes that inherit from these base classes can instantiate objects with special properties. These base classes are:
 
υπάρχουν τρία είδη ειδικών βασικών κλάσεων που χρησιμοποιούνται στον |ns3|. Οι κλάσεις που κληρονομούν από αυτές τις βασικές κλάσεις μπορούν να αρχικοποιήσουν αντικείμενα με ειδικές ιδιοότητες. Αυτές οι ειδικές κλάσεις είναι:


* class :cpp:class:`Object`
* class :cpp:class:`ObjectBase`
* class :cpp:class:`SimpleRefCount`

.. It is not required that |ns3| objects inherit from these class, but those that do get special properties. Classes deriving from class :cpp:class:`Object` get the following properties.

Δεν απαιτείται ότι τα |ns3| αντικείμενα να κληρονομούν από αυτές τις κλάσεις, αλλά εκείνες που κληρονομούν έχουν ειδικές ιδιότητες. Οι κλάσεις που προέρχονται από την κλάση class :cpp:class:`Object` αποκτούν τις παρακάτω ιδιότητες.

.. * the |ns3| type and attribute system (see :ref:`Attributes`)
* τα συστήματα είδος (type) και ιδιοτήτων (attribute) του |ns3|( βλέπε :ref:`Attributes`)
.. * an object aggregation system
* ένα σύστημα συνάθροισης αντικειμένων  
.. * a smart-pointer reference counting system (class Ptr)
* ένα σύστημα καταμέτρησης αναφοράς έξυπνων δεικτών (class Ptr) 

.. Classes that derive from class :cpp:class:`ObjectBase` get the first two properties above, but do not get smart pointers. Classes that derive from class :cpp:class:`SimpleRefCount`: get only the smart-pointer reference counting system.

Οι κλάσεις που προέρχονται από την κλάση :cpp:class:`ObjectBase` αποκτούν τις δύο πρώτες από τις παραπάνω ιδιότητες, αλλά δεν αποκτούν έξυπνους δείκτες. κλάσεις που προέρχονται από την κλαση :cpp:class:`SimpleRefCount`: αποκτούν μόνο το σύστημα καταμέτρησης αναφορών έξυπνων δεικτών.
 
.. In practice, class :cpp:class:`Object` is the variant of the three above that the |ns3| developer will most commonly encounter.

Στην πράξημ η κλάση :cpp:class:`Object` είναι η παραλλαγή των τριών παραπάνω που ένας προγραμματιστής του |ns3| θα συναντήσει συχνότερα.

.. _Memory-management-and-class-Ptr:

.. Memory management and class Ptr

Διαχείριση μνήμης και η κλάση Ptr
*********************************

.. Memory management in a C++ program is a complex process, and is often done incorrectly or inconsistently. We have settled on a reference counting design described as follows.

Η διαχείριση μνήμης σε ένα πρόγραμμα C++ είναι μία πολύπλοκη διαδικασία, και συχνά γίνεται με λάθος ή ασυνεπή τρόπο. Έχουμε καταλήξει σε ένα σχεδιασμό καταμέτρησης αναφορών όπως περιγράφεται παρακάτω.

.. All objects using reference counting maintain an internal reference count to determine when an object can safely delete itself. Each time that a pointer is obtained to an interface, the object's reference count is incremented by calling ``Ref()``. It is the obligation of the user of the pointer to explicitly ``Unref()`` the pointer when done. When the reference count falls to zero, the object is deleted.

Όλα τα αντικείμενα που χρησιμοποιούν καταμέτρηση αναφορών διατηρούν έναν εσωτερικό μετρητή αναφορών ώστε να καθορίζουν πότε το αντικείμενο μπορεί με ασφάλεια να διαγράψει τον εαυτό του. Κάθε φορά που ο δείκτης αποδίδεται σε μία διεπαφή, ο μετρητής αναφορών του αντικειμένου αυξάνεται με την κλήση της ``Ref()``. Αποτελεί υποχρέωση του χρήστη του δείκτης να τερματίσει ρητά την αναφορά του δείκτη με την κλήση της ``Unref()`` όταν έχει ολοκληρώσει την εργασία του. Όταν ο μετρητής πέσει στο μηδέν, το αντικείμενο διαγράφεται.

.. * When the client code obtains a pointer from the object itself through object   creation, or via GetObject, it does not have to increment the reference count.   
* Όταν ο κώδικας πελάτη αποκτά έναν δείκτη από το ίδιο το αντικεόμενο μέσω της δημιουργίας του αντικειμένου, ή μέσω της GetObject. δεν χρειάζεται να αυξήσει το μετρητή αναφορών.
.. * When client code obtains a pointer from another source (e.g., copying a   pointer) it must call ``Ref()`` to increment the reference count.
* Όταν ο κώδικας πελάτη, απόκτήσει ένα δείκτη από μία άλλη πηγή (π.χ. αντιγραφή ενός δείκτη) τότε πρέπει να καλέσει την ``Ref()`` ώστε να αυξήσει τον μετρητή αναφορών.
.. * All users of the object pointer must call ``Unref()`` to release the   reference.
* όλοι οι χρήστες του δείκτη του αντικειμένου πρέπει να καλέσουν την ``Unref()`` ώστε να απελευθερώσουν την αντίστοιχη αναφορά.

.. The burden for calling :cpp:func:`Unref()` is somewhat relieved by the use of the reference counting smart pointer class described below.

Το βάρος της κλήσης :cpp:func:`Unref()` ελαφρύνεται κάπως με την χρήση  της καταμέτρησης αναφορών της κλάσης έξυπνων δεικτών που περιγράφεται στην συνέχεια.

.. Users using a low-level API who wish to explicitly allocate non-reference-counted objects on the heap, using operator new, are responsible for deleting such objects.

Οι χρήστες που χρησιμοποιούν μία διεπαφή επικοινωνίας χαμηλού επιπέδου (low-level API) μπορούν να δεσμεύσουν ρητά αντικείμενα των οποίων οι αναφορές δεν καταμετρώνται και που βρίσκονται στον σωρό, χρησιμοποιώντας τον τελεστή new, είναι υπεύθυνοι για την διαγραφή αυτών των αντικειμένων. 

.. Reference counting smart pointer (Ptr)

Μετρώντας αναφορές έξυπνων δεικτών (Ptr)
++++++++++++++++++++++++++++++++++++++

.. Calling ``Ref()`` and ``Unref()`` all the time would be cumbersome, so |ns3| provides a smart pointer class :cpp:class:`Ptr` similar to :cpp:class:`Boost::intrusive_ptr`. This smart-pointer class assumes that the underlying type provides a pair of ``Ref`` and ``Unref`` methods that are expected to increment and decrement the internal refcount of the object instance.  
Η κλήση των ``Ref()`` και ``Unref()`` διαρκώς θα μπορούσε να είναι κουραστική, γι' αυτό ο |ns3| παρέχει μία κλάση έξυπνων δεικτών :cpp:class:`Ptr`, παρόμοια με την :cpp:class:`Boost::intrusive_ptr`. Αυτή η κλάση έξυπνων δεικτών υποθέτει ότι ο υποκείμενος τύπος παρέχει ένα ζεύγος μεθόδων ``Ref`` and ``Unref`` που αναμενεται να αυξάνουν και να μειώνουν τον εσωτερικό μετρητή αναφορών του στιγμιότυπο του αντικειμένου.

.. This implementation allows you to manipulate the smart pointer as if it was a normal pointer: you can compare it with zero, compare it against other pointers, assign zero to it, etc.
Η συγκεκριμένη υλοποίηση επιτρέπει την διαχείριση των έξυπνων δεικτών σαν κανονικούς δείκτες: είναι δυνατή η σύγκριση με το μηδέν, η σύγκριση με άλλους δείκτες, ανάθεση του μηδέν σε αυτούς, κτλ.

.. It is possible to extract the raw pointer from this smart pointer with the :cpp:func:`GetPointer` and :cpp:func:`PeekPointer` methods.
Είναι δυνατό να εξαχθεί ο κανονικός δείκτης από τον έξυπνο με την χρήση των μεθόδων :cpp:func:`GetPointer` και :cpp:func:`PeekPointer`.

.. If you want to store a newed object into a smart pointer, we recommend you to use the CreateObject template functions to create the object and store it in a smart pointer to avoid memory leaks. These functions are really small convenience functions and their goal is just to save you a small bit of typing.
Εάν επιθυμείτε να αποθηκεύσετε ένα νέο αντικείμενο σε έναν έξυπνο δείκτη, προτείνεται να χρησιμοποιήσετε το πρότυπο συναρτήσεων CreateObject για την δημιουργία του αντικειμένου και την αποθήκευσή του σε έναν έξυπνο δείκτη για την αποφυγή διαρροές μνήμης. Οι συγκεκριμένες συναρτήσεις είναι πολύ βολικές καθώς ο σκοπός τους είναι να μειώσουν τον χρόνο πληκτρολόγησης.  


.. CreateObject and Create
CreateObject και Create
***********************

.. Objects in C++ may be statically, dynamically, or automatically created.  This holds true for |ns3| also, but some objects in the system have some additional frameworks available. Specifically, reference counted objects are usually allocated using a templated Create or CreateObject method, as follows.
Τα αντικείμενα σε C++, μπορελι να δημιουργούνται στατικά, δυναμικά ή αυτόματα. Το γεγονός αυτό συνεχίζει να ισχύει και στην περίπτωση του |ns3|, με την διαφορά ότι κάποια αντικείμενα στο σύστημα διαθέτουν κάποια επιπλέον πλαίσια (frameworks). Συγκεκριμένα, τα αντικείμενα με μετρητή αναφορών δεσμεύονται συνήθως χρησιμοποιώντας μεθόδους πρότυπα όπως οι Create και CreateObject, όπως φαίνεται παρακάτω. 


.. For objects deriving from class :cpp:class:`Object`::
Για αντικείμενα που προέρχονται από την κλάση :cpp:class:`Object`::

    Ptr<WifiNetDevice> device = CreateObject<WifiNetDevice> ();

.. Please do not create such objects using ``operator new``; create them using :cpp:func:`CreateObject()` instead.
Παρακαλούμε μην δημιουργείται τέτοια αντικείμενα χρησιμοποιώντας τον τελεστή ``operator new``; Αντίθετα, δημιουργήστε τα χρησιμοποιώντας την συνάρτήση :cpp:func:`CreateObject()`.

.. For objects deriving from class :cpp:class:`SimpleRefCount`, or other objects that support usage of the smart pointer class, a templated helper function is available and recommended to be used::
Για αντικείμενα που προέρχονται από την κλάση  :cpp:class:`SimpleRefCount`, ή άλλα αντικείμενα που υποστηρίζουν την χρήση της κλάσης έξυπνων δεικτών, είναι διαθέσιμη μία βοηθητική κλάση πρότυπο και η οποία προτείνεται να χρησιμοποιείται::

    Ptr<B> b = Create<B> ();

.. This is simply a wrapper around operator new that correctly handles the reference counting system.
Αυτό είναι απλά ένα περιτύλιγμα (wrapper) γύρω από τον τελεστή new που χειρίζεται σωστά το σύστημα καταμέτρησης αναφορών.

.. In summary, use ``Create<B>`` if B is not an object but just uses reference counting (e.g. :cpp:class:`Packet`), and use ``CreateObject<B>`` if B derives from :cpp:class:`ns3::Object`.
Συμπερασματικά, χρησιμοποιείστε ``Create<B>`` εάν το B δεν είναι αντικείμενο, αλλά απλά χρησιμοποιεί καταμέτρηση αναφορών (π.χ :cpp:class:`Packet`), και χρησιμοποιείστε ``CreateObject<B>`` εάν το B προέρχεται από την κλάση :cpp:class:`ns3::Object`.

.. Aggregation
Συνάθροιση
***********

.. The |ns3| object aggregation system is motivated in strong part by a recognition that a common use case for |ns2| has been the use of inheritance and polymorphism to extend protocol models. For instance, specialized versions of TCP such as RenoTcpAgent derive from (and override functions from) class TcpAgent.  
Το σύστημα συνάθροισης αντικειμένν του |ns3| υποκινείται σε μεγάλο βαθμό από την παρατήρηση ότι μία συχνή περίπτωση χρήσης του |ns2| υπήρξε η χρήση της κληρονομηκότητας και του πολυμορφισμού για την επέκταση τον μοντέλων πρωτοκόλλων. Για παράδειγμα, ειδικές εκδόσεις του TCP όπως RenoTcpAgent προέρχονται από (και επανακαθορίζουν συναρτήσεις από) την κλάση TcpAgent.

.. However, two problems that have arisen in the |ns2| model are downcasts and "weak base class." Downcasting refers to the procedure of using a base class pointer to an object and querying it at run time to find out type information, used to explicitly cast the pointer to a subclass pointer so that the subclass API can be used. Weak base class refers to the problems that arise when a class cannot be effectively reused (derived from) because it lacks necessary functionality, leading the developer to have to modify the base class and causing proliferation of base class API calls, some of which may not be semantically correct for all subclasses.
Παρόλα αυτά, δύο προβλήματα έχουν προκύψει στο μοντέλο του |ns2| είναι τα downcasts και οι  "weak" βασικές κλάσεις. Ο όρος Downcasting αναφέρεται στην διαδικασία της χρήσης ενός δείκτη βασικής κλάσης σε ένα αντικείμενο και υπάρχει ερώτηση προς αυτό κατά τον χρόνο εκτέλεσης προκειμένου να βρεθούν πληροφορίες σχετικά με τον τύπο του, και χρησιμοποιείται για την ρητή μετατροπή (cast) του δείκτη σε δείκτη υποκλάσης ώστε να μπορεί να χρησιμοποιηθεί η διεπαφή επικοινωνίας(API) της υποκλάσης. Ο όρος weak βασική κλάση αναφέρεται στα προβλήματα που προκύπτουν όταν μία κλάση δεν μπορεί να απαναχρησιμοποιηθεί αποδοτικά (derived from) διότι δεν διαθέτει την απαραίτητη λειτουργικότητα, αναγκάζοντας τον χρήστη να τροποποιήσει την βασική κλάση και να πολλαπλασιάσει τις κλήσεις της διεπαφής επικοινωνίας (API) της βασικής κλάσης, μερικές από τις οποίες μπορεί να μην είναι σημασιολογικά σωστές για όλες τις υποκλάσεις. 

.. |ns3| is using a version of the query interface design pattern to avoid these problems. This design is based on elements of the `Component Object Model <http://en.wikipedia.org/wiki/Component_Object_Model>`_ and `GNOME Bonobo <http://en.wikipedia.org/wiki/Bonobo_(component_model)>`_ although full binary-level compatibility of replaceable components is not supported and we have tried to simplify the syntax and impact on model developers.  

Ο |ns3| χρησιμοποιεί μία εκδοση του σχεδιαστικού πρότυπου διεπαφής ερωτημάτων ώστε να αποφύγει τέτοιου είδους προβλήματα. Ο σχεδιασμός αυτός βασίζεται σε στοιχεία του `Component Object Model <http://en.wikipedia.org/wiki/Component_Object_Model>`_ και `GNOME Bonobo <http://en.wikipedia.org/wiki/Bonobo_(component_model)>`_  παρόλο που πλήρης συμβατότητα των επιμέρους τμημαάτων σε δυαδικό επίπεδο δεν υποστηρίζεται και έχει γίνει προσπάθεια να απλοποιηθει το συνακτικό και το αντίκτυπο στους προγραμματιστές του μοντέλου.   

.. Examples
Παραδείγματα
************

.. Aggregation example
Παράδειγμα συνάθροισης
++++++++++++++++++++++

.. :cpp:class:`Node` is a good example of the use of aggregation in |ns3|.  Note that there are not derived classes of Nodes in |ns3| such as class :cpp:class:`InternetNode`.  Instead, components (protocols) are aggregated to a node. Let's look at how some Ipv4 protocols are added to a node.::

Η κλάση :cpp:class:`Node` είναι ένα καλό παράδειγμα χρήσης της συνάθροισης στον |ns3|. Να σημειωθεί ότι δεν υπάρχουν κλάσεις που προέρχονται από Nodes στον |ns3| όπως η κλάση :cpp:class:`InternetNode`. Αντί για αυτό,  συστατικα -components (πρωτόκολα) σύναθροίζοντται σε ένα κόμβο. Στην συνέχεια εξετάζουμε πως μερικά πρωτόκολλα Ipv4 προστίθενται ώστε να σχηματίσουν ένα κόμβο.::

    static void
    AddIpv4Stack(Ptr<Node> node)
    {
      Ptr<Ipv4L3Protocol> ipv4 = CreateObject<Ipv4L3Protocol> ();
      ipv4->SetNode (node);
      node->AggregateObject (ipv4);
      Ptr<Ipv4Impl> ipv4Impl = CreateObject<Ipv4Impl> ();
      ipv4Impl->SetIpv4 (ipv4);
      node->AggregateObject (ipv4Impl);
    }

.. Note that the Ipv4 protocols are created using :cpp:func:`CreateObject()`. Then, they are aggregated to the node. In this manner, the Node base class does not need to be edited to allow users with a base class Node pointer to access the Ipv4 interface; users may ask the node for a pointer to its Ipv4 interface at runtime. How the user asks the node is described in the next subsection.
Να σημειωθεί ότι τα πρωτόκολλα δημιουργούνται χρησιμοποιώντας :cpp:func:`CreateObject()`. Τότε, συναθροίζονται σε ένα κόμβο. Με αυτό τον τρόπο η βασική κλάση Node δεν χρειάζεται να τροποποιηθεί ώστε να επιτρέψει σε χρήστες μέ έναν δείκτη της βασικής κλάσης Node να έχουν πρόσβαση στην διεπαφή Ipv4. Οι χρήστες να ρωτήσουν τον κόμβο για έναν δείκτη προς την διεπαφή Ipv4 κατά τον χρόνο εκτέλεσης. Ο τρόπος που ο χρήστης πραγματοποιεί αυτή την ερώτηση περιγράφεται στην επόμενη υποενότητα.

.. Note that it is a programming error to aggregate more than one object of the same type to an :cpp:class:`ns3::Object`. So, for instance, aggregation is not an option for storing all of the active sockets of a node.
Να σημειωθεί ότι είναι προγραμματιστικό λάθος να συναθροίζονται περισσότερα από ενα αντικείμενο του ίδιου τύπου σε ένα αντικείμενο :cpp:class:`ns3::Object`. Έτσι για παράδειγμα, η συνάθροιση δεν αποτελεί επιλογή για την αποθήκευση όλων των ενεργών socket ενός κόμβου. 

.. GetObject example
Το παράγειγμα GetObject
+++++++++++++++++++++++

.. GetObject is a type-safe way to achieve a safe downcasting and to allow interfaces to be found on an object.  
Η μέθοδος GetObject αποτελεί μία ασφαλή μέθοδο όσο αφορά την μετατροπή τύπων δεδομένων (type-safe) ώστε να επιτευχθεί με ασφαλή τρόπο downcasting και να επιτραπεί σε διεπαφές να είναι ορατές σε ένα αντικείμενο.

.. Consider a node pointer ``m_node`` that points to a Node object that has an implementation of IPv4 previously aggregated to it. The client code wishes to configure a default route. To do so, it must access an object within the node that has an interface to the IP forwarding configuration. It performs the following::
Θεωρείστε ένα δείκτη σε κόμβο, έστω ``m_node``, που δείχνει σε ένα αντικείμενο Node στο οποίο προηγουμένως έχει συναθροιστεί μία υλοποίηση IPv4. Ο κώδικας πελάτη επιθυμεί να τροποποιήσει την προκαθορισμένη διαδρομή. Για να το επιτύχει, θα πρέπει να έχει πρόσβαση σε ένα αντικείμενο στο εσωτερικό του κόμβου το οποίο διαθέτει μία διεπαφή προς το ρύθμιση προώθησης IP. θα εκτελέσει τα ακόλουθα::


    Ptr<Ipv4> ipv4 = m_node->GetObject<Ipv4> ();

.. If the node in fact does not have an Ipv4 object aggregated to it, then the method will return null. Therefore, it is good practice to check the return value from such a function call. If successful, the user can now use the Ptr to the Ipv4 object that was previously aggregated to the node.
Εάν ο κόμβος στην ουσία δεν έχει ένα Ipv4 αντικέιμενο συναθροισμένο σε αυτόν, τότε η μέθοδος θα επιστρεψει null. Συνεπώς, αποτελεί καλή πρακτική, να γίνεται έλεγχος της επιστρεφόμενης τιμής της κλήσης μία συνάρτησης σαν και αυτή. Εάν είναι επιτυχής, ο χρήστης μπορεί να χρησιμοποιήσει τον δείκτη Ptr προς το αντικείμενο Ipv4, το οποίο έχει συναθροιστεί προηγουμένως με το κόμβο.

.. Another example of how one might use aggregation is to add optional models to objects. For instance, an existing Node object may have an "Energy Model" object aggregated to it at run time (without modifying and recompiling the node class). An existing model (such as a wireless net device) can then later "GetObject" for the energy model and act appropriately if the interface has been either built in to the underlying Node object or aggregated to it at run time.  However, other nodes need not know anything about energy models.
Ένα άλλο παράδειγμα για το πώς κάποιος μπορεί να χρησιμοποιήσει την συνάθροιση είναι την προσθήκη προαιρετικών μοντέλων σε αντικείμενα. Για παράδειγμα, ένα αντικείμενο Node μπορεί να έχει ένα αντικείμενο "Energy Model" συναθροισμένο σε αυτό κατά τον χρόνο εκτέλεσης (χωρίς να τροποποιηθεί ή να μεταγλωττιστεί εξαρχής η κλάση κόμβος). Ένα ήδη υπάρχον μοντέλο (όπως μία ασύρματη συσκευή δικτύου) μπορεί αργότερα να καλέσει την "GetObject" του μοντέλου ενέργειας (energy model) και να δράσει κατάλληλα εάν η διεπαφή είτε είναι δημιουργηθεί στο υποκείμενο αντικείμενο Node ή είχε συναθροιστεί σε αυτό κατά τον χρόνο εκτέλεσης. Παρόλα αυτά, άλλοι κόμβοι δεν χρειάζεται να γνωρίζουν οτιδήποτε σχετικά με τα μοντέλα ενέργειας.

.. We hope that this mode of programming will require much less need for developers to modify the base classes.
Ελπίζουμε ότι ο συγκεκριμένος τρόπος προγραμματισμού θα μειώσει τις απαιτήσεις των προγραμματιστών για τροποποιήσεις στις βασικές κλάσεις.

.. Object factories
Εργοστάσια αντικειμένων
***********************

.. A common use case is to create lots of similarly configured objects. One can repeatedly call :cpp:func:`CreateObject` but there is also a factory design pattern in use in the |ns3| system. It is heavily used in the "helper" API. 
Μια συνηθισμένη περίπτωση χρήσης είναι η δημιουργία αντικειμένων που έχουν ρυθμιστεί με παρόμοιο τρόπο. Κάποιος μπορεί να καλεσει κατ' επανάληψη την :cpp:func:`CreateObject` αλλά υπάρχει επίσης ένα εργοστασιακό σχεδιαστικό πρότυπο σε χρήση στο σύστημα του |ns3|. Χρησιμοποιείται ευρέως στην διεπαφή επικοινωνίας "helper".

.. Class :cpp:class:`ObjectFactory` can be used to instantiate objects and to configure the attributes on those objects::
Η κλάση :cpp:class:`ObjectFactory` μπορεί να χρησιμοποιηθεί για την αρχικοποίηση αντικειμένων και να ρυθμισει τις ιδιότητες σε εκείνα τα αντικείμενα::

    void SetTypeId (TypeId tid);
    void Set (std::string name, const AttributeValue &value);
    Ptr<T> Create (void) const;

.. The first method allows one to use the |ns3| TypeId system to specify the type of objects created. The second allows one to set attributes on the objects to be created, and the third allows one to create the objects themselves. 
Η πρώτη μέθοδος επιτρέπει την χρήση του συστήματος TypeId του |ns3| για τον καθορισμό του είδους (type) των αντικειμένων που δημιουργούνται. Η δεύτερη επιτρέπει τον ορισμό των ιδιοτήτων των αντικειμένων που πρόκειται να δημιουργηθούν,  και η που τρίτη επιτρέπει την δημιουργία των ίδιων των αντικειμένων.

.. For example: ::
Για παράδειγμα: ::

    ObjectFactory factory;
    // Make this factory create objects of type FriisPropagationLossModel
    factory.SetTypeId ("ns3::FriisPropagationLossModel")
    // Make this factory object change a default value of an attribute, for
    // subsequently created objects
    factory.Set ("SystemLoss", DoubleValue (2.0));
    // Create one such object
    Ptr<Object> object = factory.Create (); 
    factory.Set ("SystemLoss", DoubleValue (3.0));
    // Create another object with a different SystemLoss
    Ptr<Object> object = factory.Create (); 

Downcasting
***********

.. A question that has arisen several times is, "If I have a base class pointer (Ptr) to an object and I want the derived class pointer, should I downcast (via C++ dynamic cast) to get the derived pointer, or should I use the object aggregation system to :cpp:func:`GetObject\<> ()` to find a Ptr to the interface to the subclass API?"
Μία ερώτηση που προκύπτει αρκετές φορές είναι, "εάν έχω έναν δείκτη(Ptr) της βασικής κλάσης προς ένα αντικείμενο και θέλω έναν δείκτη προς την παραγόμενη κλάση, είναι προτιμότερο να εφαρμόσω downcast (μέσω C++ δυναμικής μετατροπής) ώστε να λάβω τον δείκτη της παραγόμενης κλάσης ή θα ήταν καλύτερα να χρησιμοποιήσω το σύστημα συνάθροισης αντικειμένων :cpp:func:`GetObject\<> ()` ώστε να βρω έναν Ptr προς την διεπαφή του API της υποκλάσης;"

.. The answer to this is that in many situations, both techniques will work. |ns3| provides a templated function for making the syntax of Object dynamic casting much more user friendly::
Η απάντηση σε αυτό είναι ότι σε πολλές περιπτώσεις, και οι δύο τεχνικές θα λειτουργήσουν. Ο |ns3| παρέχει ένα πρότυπο συναρτήσεων ώστε να κάνει πολύ πιο φιλική προς τον χρήστη την συνταξη μετατροπή αντικειμενων (Object dynamic casting)::

    template <typename T1, typename T2>
    Ptr<T1>
    DynamicCast (Ptr<T2> const&p)
    {
      return Ptr<T1> (dynamic_cast<T1 *> (PeekPointer (p)));
    }

.. DynamicCast works when the programmer has a base type pointer and is testing against a subclass pointer. GetObject works when looking for different objects aggregated, but also works with subclasses, in the same way as DynamicCast. If unsure, the programmer should use GetObject, as it works in all cases. If the programmer knows the class hierarchy of the object under consideration, it is more direct to just use DynamicCast.
Η δυαμική μετατροπή(DynamicCast) λειτουργεί όταν ο προγραμματιστής έχει έναν δείκτη βασικής κλάσης και δοκιμάζει ενάντια σε έναν δείκτη υποκλάσης. Η GetObject λειτουργεί όταν ενδιαφερόμαστε για διαφορετικά αντικείμενα που έχουν συναθροιστεί, αλλά λειτουργεί επίσης με υποκλάσεις, με τον ίδιο τρόπο σαν DynamicCast. Εάν ο προγραμματιστής δεν είναι σίγουρος μπορεί να χρησιμοποιήσει την GetObject καθώς λειτουργεί σε όλες τις περιπτώσεις. Εάν ο προγραμματιστής γνωρίζει την ιεραρχία κλάσεων του αντικειμένου ενδιαφέροντος, είναι πιο άμεσο να χρησιμοποιήσει απλά DynamicCast. 
 

