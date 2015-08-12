.. include:: replace.txt
.. highlight:: cpp

.. Callbacks 
Επανακλήσεις
------------

.. Some new users to |ns3| are unfamiliar with an extensively used programming idiom used throughout the code: the *ns-3 callback*. This chapter provides some motivation on the callback, guidance on how to use it, and details on its implementation.
Κάποιοι νέοι χρήστες του |ns3| δεν είναι εξοικειωμένοι με ένα προγραμματιστικό ιδίωμα που χρησιμοποιείται ευρέως στον κώδικα: το *ns-3 callback*. Η συγκεκριμένη ενότητα παρέχει κάποια κίνητρα σχετικά με τις επανακλήσεις (callback), καθοδήγηση για τον τρόπο χρησιμοποίησής τους και λεπτομέριες για την υλοποίησή τους.

.. Callbacks Motivation
Κίνητρα για επανακλήσεις
************************

.. Consider that you have two simulation models A and B, and you wish to have them pass information between them during the simulation. One way that you can do that is that you can make A and B each explicitly knowledgeable about the other, so that they can invoke methods on each other::
Θεωρήστε ότι έχετε δύο μοντέλα προσομοίωησης A και B και επιθυμείτε να περάσετε πληροφορία μέσω αυτών κατά την διάρκεια μία προσομοίωσης. Ένας τρόπος για να το επιτύχετε είναι κάνετε το Α και Β να αποκτήσουν γνώση το ένα για το άλλο ρητά, ώστε να αποκτήσουν την δυνατότητα καλούν μεθόδους το ένα στο άλλο::   

  class A {
  public:
    void ReceiveInput ( // parameters );
    ...
  }

   (in another source file:)
  
  class B {
  public:
    void DoSomething (void);
    ...

  private:
   
    A* a_instance; // Δείκτης σε ένα Α
  }

  void
  B::DoSomething()
  {
    .. // Tell a_instance that something happened
    //Ενημερώστε a_instance ότι κάτι συνέβη
    a_instance->ReceiveInput ( // parameters);
    ...
  }

.. This certainly works, but it has the drawback that it introduces a dependency on A and B to know about the other at compile time (this makes it harder to have independent compilation units in the simulator) and is not generalized; if in a later usage scenario, B needs to talk to a completely different C object, the source code for B needs to be changed to add a ``c_instance`` and so forth. It is easy to see that this is a brute force mechanism of communication that can lead to programming cruft in the models.  
Αυτό φυσικά λειτουργεί, αλλά έχει το μειονέκτημα ότι εισάγει μία εξάρτηση μεταξύ των Α και Β σχετικά με το ότι θα πρέπει να γνωρίζονται μεταξύ τους κατά τον χρόνο μεταγλώττισης. (αυτό δυσκολεύει την ύπαρξη ανεξάρτητων μονάδων μεταγλώττιση κώδικα στον προσομοιωτή) και δεν γενικεύονται. Εάν σε ένα διαφορετικό σενάριο χρήσης  ο B χρειάζεται να επικοινωνήσει με ένα εντελώς διαφορετικό αντικείμενο C, ο πηγαίος κώδικας του B απαιτείται να αλλάξει ώστε να προστεθεί ένα ``c_instance`` στιγμιότυπο και ούτω καθεξής. Είναι εύκολο να διαπιστώσουμε ότι ο συγκεκριμένος είναι ένας ισχυρός αλλά άκομψος μηχανισμός επικοινωνίας που μπορεί να οδηγήσει σε αχρείαστα πολύπλοκα μοντέλα προγραμματισμού.

.. This is not to say that objects should not know about one another if there is a hard dependency between them, but that often the model can be made more flexible if its interactions are less constrained at compile time.
Αυτή η παρατήρηση δεν έχει ως σκοπό να υποστηρίξει ότι τα αντικείμενα δεν πρέπει να έχουν γνώση το ένα για την ύπαρξη του άλλου, εάν υπάρχει ισχυρή αλληλεξάρτηση μεταξύ τους, αλλά ότι το μοντέλο μπορεί να γίνει περισσότερο ελαστικό εάν η αλληλεξάρτηση είναι λιγότερο περιοριστική κατά τον χρόνο μεταγλώττισης

.. This is not an abstract problem for network simulation research, but rather it has been a source of problems in previous simulators, when researchers want to extend or modify the system to do different things (as they are apt to do in research). Consider, for example, a user who wants to add an IPsec security protocol sublayer between TCP and IP::
Αυτό δεν είναι ένα αόριστα διατυπωμένο πρόβλημα για την έρευνα σχετικά με την προσομοίωση δικτύων, αλλά μάλλον είναι μία πηγή προβλημάτων σε προηγούμενους προσομοιωτές, όταν ερευνητές επιθυμούν να επεκτείνουν ή να τροποποιήσουν το σύστημα ώστε να εκτελεί και διαφορετικές λειτουργίες (όπως θα έπρεπε να κάνουν κατά την έρευνά τους). Θεωρείστε, για παράδειγμα έναν χρήστη που επιθυμεί να προσθέσει έναν πρωτοκολλο ασφάλειας IPsec μεταξύ TCP και IP::


  ------------                   -----------
  |   TCP    |                   |  TCP    |
  ------------                   -----------
       |           becomes ->        |
  -----------                    -----------
  |   IP    |                    | IPsec   |
  -----------                    -----------
                                     |
                                 -----------
                                 |   IP    |
                                 -----------


.. If the simulator has made assumptions, and hard coded into the code, that IP always talks to a transport protocol above, the user may be forced to hack the system to get the desired interconnections. This is clearly not an optimal way to design a generic simulator.
Εάν ο προσομοιωτής κάνει θεωρήσεις, και περιέχει κλειστά κομμάτια κώδικα, ότι μία IP επικοινωνεί πάντα μέσω ενός διαφανούς πρωτοκόλλου, ο χρήστης μπορεί να αναγκαστεί να παραβιάζει το σύστημα ώστε να έχει πρόσβαση στις επιθυμητές διασυνδέσεις. Αυτός προφανώς δεν είναι ο βέλτιστος τρόπος σχεδιασμού ενός γενικού προσομοιωτή.


.. Callbacks Background
Παρασκήνιο Επανακλήσεων
********************

.. .. note:: Readers familiar with programming callbacks may skip this tutorial section.
.. note:: Οι Αναγνώστες που έχουν εξοικειωθεί με τις προγραμματιστικές επανακλήσεις μπορούν να προσπεράσουν την συγκεκριμένη ενότητα.

.. The basic mechanism that allows one to address the problem above is known as a *callback*. The ultimate goal is to allow one piece of code to call a function (or method in C++) without any specific inter-module dependency.
Ο βασικός μηχανισμός που επιτρέπει την αντιμετώπιση του προαναφερθέντος προβλήματος είναι γνωστός ως *επανάκληση(callback)*. Ο τελικός στόχος είναι να επιτραπεί ένα κομμάτι κώδικα να καλέσει μία συνάρτηση (ή C++ μέθοδο) χωρίς κάποια εξάρτηση μεταξύ μονάδων.

.. This ultimately means you need some kind of indirection -- you treat the address of the called function as a variable.  This variable is called a pointer-to-function variable. The relationship between function and pointer-to-function pointer is really no different that that of object and pointer-to-object.
Αυτό σημαίνει ότι, τελικά, είναι απαραίτητη κάποιας μορφής ανακατεύθυνση(indirection) - που αντιμετωπίζει τη διεύθυνση της κληθείσας συνάρτησης ως μια μεταβλητή. Αυτή η μεταβλητή ονομάζεται μεταβλητή δείκτης-σε-συνάρτηση( pointer-to-function variable). Η σχέση μεταξύ συνάρτησης και του δείκτη-σε-συνάρτηση είναι ανάλογη με την σχέση αντικείμενο και δείκτη-σε-αντικείμενο.


.. In C the canonical example of a pointer-to-function is a pointer-to-function-returning-integer (PFI). For a PFI taking one int parameter, this could be declared like,
Στην C το κανονικό παράδειγμα ενός δείκτη-σε-συνάρτηση είναι ένας δείκτης-σε-συνάρτηση-που-επιστρέφει-ακέραιο(PFI). Για έναν PFI που δέχεται ένα ακέραιο όρισμα, η αντίστοιχη δήλωση έχει ως εξής::

  int (*pfi)(int arg) = 0;

.. What you get from this is a variable named simply ``pfi`` that is initialized to the value 0. If you want to initialize this pointer to something meaningful, you have to have a function with a matching signature. In this case::
Το αποτέλεσμα της παραπάνω δήλωσης είναι μία μεταβλητή που ονομάζεται απλά  ``pfi`` και η οποία αρχικοποιείται με την τιμή 0. Εάν επιθυμείτε να αρχικοποιήσετε τον δείκτη με μία τιμή που να έχει νόημα, πρέπει να είναι διαθέσιμη μία συνάρτηση με υπογραφή (ορίσματα - επιστρεφόμενο τύπο) που να ταιριάζει. Σε αυτή την περίπτωση::

  int MyFunction (int arg) {}

.. If you have this target, you can initialize the variable to point to your function like::
Εάν επιθυμείτε ο δείκτης να έχει στόχο την παραπάνω συνάρτηση, μπορείτε να αρχικοποιήσετε την μεταβλητή pfi ώστε να δείχνει στην συγκεκριμένη συνάρτηση ως εξής:: 

  pfi = MyFunction;

.. You can then call MyFunction indirectly using the more suggestive form of the call::
Στην συνέχεια μπορείτε να καλείτε την συνάρτηση MyFunction έμμεσα χρησιμοποιώντας την περισσότερο υποδηλωτική μορφή της κλήσης της συνάρτησης:: 

  int result = (*pfi) (1234);

.. This is suggestive since it looks like you are dereferencing the function pointer just like you would dereference any pointer. Typically, however, people take advantage of the fact that the compiler knows what is going on and will just use a shorter form::
Αυτή η μορφή είναι υποδηλωτική, δεδομένου ότι φαίνεται σαν πραγματοποιείται αποαναφοροποίηση του δείκτη συνάρτησης ακριβώς με τον ίδιο τρόπο που θα αποαναφοροποιούσαμε κάθε δείκτη. Συνήθως, όμως, οι άνθρωποι επωφελούνται από το γεγονός ότι ο μεταγλωττιστής γνωρίζει τι συμβαίνει και γι αυτό προτιμούν να χρησιμοποιήσουν μια συντομότερη μορφή ::

  int result = pfi (1234);

.. Notice that the function pointer obeys value semantics, so you can pass it around like any other value. Typically, when you use an asynchronous interface you will pass some entity like this to a function which will perform an action and *call back* to let you know it completed. It calls back by following the indirection and executing the provided function.
Παρατηρήστε ότι ο δείκτης συνάρτησης μπορεί να ερμηνευθεί ως τιμή (υπακούει στην σημασίολογία της τιμής), ώστε να μπορείτε να την χρησιμοποιήσετε σαν οποιαδήποτε άλλη τιμή. Συνήθως, όταν χρησιμοποιείτε μια ασύγχρονη διεπαφή θα περάσετε κάποια οντότητα όπως αυτό στην συνάρτηση που θα εκτελέσει μια ενέργεια και *επανακαλέσει (call back)* ώστε για να ενημερώσει ότι ολοκληρώθηκεη εκτέλεσή της. Θα επανακαλέσει, ακολουθώντας την ανακατεύθυνση και εκτελώντας την παρεχόμενη συνάρτηση.

.. In C++ you have the added complexity of objects. The analogy with the PFI above means you have a pointer to a member function returning an int (PMI) instead of the pointer to function returning an int (PFI).

Στη C++ εμφανίζεται η επιπρόσθετη πολυπλοκότητα των αντικειμένων. Η αναλογία με τους PFI που παρουσιάστηκαν παραπάνω, συνεπάγεται ότι έχετε ένα δείκτη σε μια συνάρτηση-μέλος που επιστρέφει έναν int (PMI) αντί του δείκτη σε συνάρτηση που επιστρέφει έναν int (PFI).

.. The declaration of the variable providing the indirection looks only slightly different::
Η δήλωση της μεταβλητής που παρέχει την ανακατεύθυνση μοιάζει ελαφρώς διαφορετική ::

  int (MyClass::*pmi) (int arg) = 0;

.. This declares a variable named ``pmi`` just as the previous example declared a variable named ``pfi``. Since the will be to call a method of an instance of a particular class, one must declare that method in a class::
Η παραπάνω κώδικας δηλώνει μια μεταβλητή με το όνομα ``pmi`` ακριβώς όπως το προηγούμενο παράδειγμα δηλώθηκε μια μεταβλητή με όνομα  ``pfi``. Δεδομένου ότι ο σκοπός είναι να κληθεί μια μέθοδος ενός στιγμιοτύπου μιας συγκεκριμένης κλάσης, πρέπει κανείς να δηλώσει τη μέθοδο αυτή, σε μία κλάση, ως εξής::

  class MyClass {
  public:
    int MyMethod (int arg);
  };

.. Given this class declaration, one would then initialize that variable like this::
Δεδομένης αυτής της δήλωσης για την κλάσης, κάποιος θα μπορούσε να αρχικοποιήσει αυτή τη μεταβλητή, με αυτό τον τρόπο::

  pmi = &MyClass::MyMethod;

.. This assigns the address of the code implementing the method to the variable, completing the indirection. In order to call a method, the code needs a ``this`` pointer. This, in turn, means there must be an object of MyClass to refer to. A simplistic example of this is just calling a method indirectly (think virtual function)::
Το παραπάνω παράδειγμα εκχωρεί τη διεύθυνση του κώδικα που υλοποιεί την μέθοδο στη μεταβλητή, ολοκληρώνοντας την ανακατεύθυνση. Για να καλέσετε μια μέθοδο, ο κώδικας χρειάζεται ένα δείκτη ``this`` . Αυτό, με τη σειρά του, σημαίνει ότι πρέπει να υπάρχει ένα αντικείμενο της κλάσης MyClass στο οποίο ο δείκτης this αναφέρεται. Ένα απλοϊκό παράδειγμα είναι απλά, η έμμεση κλήση μιας μεθόδου (σκεφτείτε εικονική συνάρτηση-virtual function) ::

  int (MyClass::*pmi) (int arg) = 0;  // Δήλωση ενός PMI
  pmi = &MyClass::MyMethod;           // Δείξε προς τον κώδικα υλοποίησης
  
  MyClass myClass;                    // Απαιτείται ένα στιγμιότυπο της κλάσης
  (myClass.*pmi) (1234);              // κάλεσε την μέθοδο με έναν δείκτη αντικειμένου
  
.. int (MyClass::*pmi) (int arg) = 0;  // Declare a PMI
.. pmi = &MyClass::MyMethod;           // Point at the implementation code

.. MyClass myClass;                    // Need an instance of the class
.. (myClass.*pmi) (1234);              // Call the method with an object ptr
 
  
  

.. Just like in the C example, you can use this in an asynchronous call to another module which will *call back* using a method and an object pointer. The straightforward extension one might consider is to pass a pointer to the object and the PMI variable. The module would just do::
Ακριβώς όπως στο παράδειγμα της C, μπορείτε να το χρησιμοποιήσετε σε μια ασύγχρονη κλήση προς μία άλλη μονάδα που θα *επανακαλέσει* χρησιμοποιώντας μια μέθοδο και ένα δείκτη αντικειμένου. Θα μπορούσε κάποιος να σκεφτεί, ότι η απευθείας επέκταση είναι να περάσουμε έναν δείκτη στο αντικείμενο και στην PMI μεταβλητή. Η μονάδα θα κάνει ακριβώς ::

  (*objectPtr.*pmi) (1234);

.. to execute the callback on the desired object.
για να εκτελέσει την επανάκληση στο επιθυμητό αντικείμενο.

.. One might ask at this time, *what's the point*? The called module will have to understand the concrete type of the calling object in order to properly make the callback. Why not just accept this, pass the correctly typed object pointer and do ``object->Method(1234)`` in the code instead of the callback?  This is precisely the problem described above. What is needed is a way to decouple the calling function from the called class completely. This requirement led to the development of the *Functor*.
Κάποιος θα μπορούσε να ρωτήσει, *και ποιο το νόημα*; Η μονάδα που καλείται πρέπει να μπορεί να αντιληφθεί το συγκεκριμένο τύπο του αντικειμένου που καλείται, ώστε να είναι σε θέση να επανακαλέσει με τον κατάλληλο τρόπο. Γιατί να μην το αποδεχτούμε αυτό απλά και να περάσουμε τον δείκτη αντικειμένου με τον καταλληλο τύπο, χρησιμοποιώντας την ``object->Method(1234)`` αντί της επανάκλησης; Αυτό είναι ακριβώς το πρόβλημα που περιγράψαμε παραπάνω. Αυτο που χρειάζεται είναι ένας τρόπος να ξεμπλέξουμε την συνάρτηση που καλεί από την κλάση που καλείται εντελώς. Αυτή η απαίτηση οδηγεί στην ανάπτυξη του *Functor*.

.. A functor is the outgrowth of something invented in the 1960s called a closure. It is basically just a packaged-up function call, possibly with some state.  
Ο functor προέκυψε σαν παραπροϊόν από κάτι που εφευρέθηκε στην δεκαετία του 1960 ονομάζεται το κλείσιμο(closure). Πρόκειται ουσιαστικά για μία πακεταρισμένη κληση μίας συνάρτησης, ενδεχομένως μαζί με κάποια κατάσταση(state).

.. A functor has two parts, a specific part and a generic part, related through inheritance. The calling code (the code that executes the callback) will execute a generic overloaded ``operator ()`` of a generic functor to cause the callback to be called. The called code (the code that wants to be called back) will have to provide a specialized implementation of the ``operator ()`` that performs the class-specific work that caused the close-coupling problem above.  
Ένας functor έχει δύο μέλη, ένα συγκεκριμένο μέρος και ένα γενικό μέρος, που σχετίζονται μέσω κληρονομικότητας. Ο κώδικά που καλεί (και εκτελεί την επανάκληση) θα εκτελέσει έναν γενικό υπερφορτωμένο ``operator ()`` του γενικού functor ώστε να προκαλέσει την κλήση την επανάκλησης. Ο κώδικάς που καλείται (ο κώδικάς που είναι επιθυμητό να επανακληθεί) θα πρέπει παράχει μία εξειδικευμένη υλοποίηση του ``operator ()`` που εκτελεί την εργασία που σχετίζεται με την κλάση( class-specific) που προκάλεσε το πρόβλημα κλεισίματος της σύνδεσης(close-coupling) που περιγράψαμε προηγουμένως

.. With the specific functor and its overloaded ``operator ()`` created, the called code then gives the specialized code to the module that will execute the callback (the calling code).
Με τον συγκεκριμένο functor και τον υπερφορτωμένο του ``operator ()`` που δημιουργήθηκε, το κώδικας που καλείται δίνει στην συνέχεια τον εξειδικευμένο κώδικα στην μονάδα που θα εκτελέσει την επανάκληση (τον κώδικα που καλεί).

.. The calling code will take a generic functor as a parameter, so an implicit cast is done in the function call to convert the specific functor to a generic functor.  This means that the calling module just needs to understand the generic functor type. It is decoupled from the calling code completely.
Ο κώδικας που καλεί θα πάρει έναν γενικό functor σαν παράμετρο, ούτως ώστε να πραγματοποιηθεί κατά την κλήση της συνάρτησης μία έμμεση μετατροπή που να μετατρέψει τον ειδικό functor σε γενικό functor. Αυτό συμαίνει ότι η μονάδα που καλεί, χρειάζεται μόνο να καταλάβει τον τύπο του γενικού functor. Ξεμπλέκεται από τον κώδικα που καλεί εντελώς.


.. The information one needs to make a specific functor is the object pointer and the pointer-to-method address. 
Η απαραίτητη πληροφορία για την δημιουργία ένος ειδικού functor είναι ο δείκτης αντικειμένου και η διεύθυνση του δείκτη-σε-μέθοδο.

.. The essence of what needs to happen is that the system declares a generic part of the functor::
Η ουσία του τι πρέπει να συμβεί είναι ότι το σύστημα δηλώνει ένα γενικό μέρος του functor ::

  template <typename T>
  class Functor
  {
  public:
    virtual int operator() (T arg) = 0;
  };

.. The caller defines a specific part of the functor that really is just there to implement the specific ``operator()`` method::
Αυτός που καλεί ορίζει ενα συγκεκριμένο μέρος του functor που θα υλοποιήσει την συγκεκριμένη μέθοδο ``operator()`` ::

  template <typename T, typename ARG>
  class SpecificFunctor : public Functor<ARG>
  {
  public:
    SpecificFunctor(T* p, int (T::*_pmi)(ARG arg))
    {
      m_p = p;
      m_pmi = _pmi;
    }

    virtual int operator() (ARG arg)
    {
      (*m_p.*m_pmi)(arg);
    }
  private:
    int (T::*m_pmi)(ARG arg);
    T* m_p;
  };

.. Here is an example of the usage::
Ακολουθεί ένα παράδειγμα χρήσης ::

  class A
  {
  public:
  A (int a0) : a (a0) {}
  int Hello (int b0)
  {
    std::cout << "Hello from A, a = " << a << " b0 = " << b0 << std::endl;
  }
  int a;
  };

  int main()
  {
    A a(10);
    SpecificFunctor<A, int> sf(&a, &A::Hello);
    sf(5);
  }

.. The previous code is not real ns-3 code.  It is simplistic example  code used only to illustrate the concepts involved and to help you understand the system more.  Do not expect to find this code anywhere in the ns-3 tree.

.. note:: Ο προηγούμενος κώδικας δεν είναι πραγματικός κώδικας ns-3. Είναι ένα απλουστευμένο παράδειγμα κώδικα που χρησιμοποιείται μόνο για την παρουσίαση των εννοιών που εμπλέκονται και να σας βοηθήσει να καταλάβετε καλύτερα το σύστημα. Μην περιμένετε να βρείτε αυτόν τον κώδικα οπουδήποτε στην ιεραρχία του ns-3.

.. Notice that there are two variables defined in the class above.  The m_p variable is the object pointer and m_pmi is the variable containing the address of the function to execute.
Παρατηρήστε ότι υπάρχουν δύο μεταβλητές που ορίζονται την παραπάνω κλάση. Η μεταβητή m_p είναι ο δείκτης αντικειμένου και η μεταβλητή m_pmi έιναι η μεταβλητή που περιέχει την διεύθυνση της συνάρτησης που πρόκειται να εκτελεστεί.


.. Notice that when ``operator()`` is called, it in turn calls the method provided with the object pointer using the C++ PMI syntax.
Παρατηρήστε ότι όταν ο ``operator()`` καλείται, καλεί με την σειρά του την μέθοδο που παρέχεται με τον δείκτη αντικειμένου χρησμιμοποιώντας την στην C++ PMI σύνταξη.

.. To use this, one could then declare some model code that takes a generic functor as a parameter::
Για να το χρησιμοποιηθεί, κάποιος θα μπορούσε να δηλώσει ένα μοντέλο κώδικα όποθ θα έπερνε ένα γενικό functor σαν παράμετρο::


  void LibraryFunction (Functor functor);

.. The code that will talk to the model would build a specific functor and pass it to ``LibraryFunction``:: 
Ο κώδικας που θα επικοινωνούσε με το μοντέλο θα έπρεπε να δημιουργήσει ένα εξειδικευμένο functor και να το περάσει στην συνάρτηση ``LibraryFunction``::

  MyClass myClass;
  SpecificFunctor<MyClass, int> functor (&myclass, MyClass::MyMethod);

.. When ``LibraryFunction`` is done, it executes the callback using the ``operator()`` on the generic functor it was passed, and in this particular case, provides the integer argument
Όταν όλοκληρωθεί η  ``LibraryFunction`` εκτελείται η επανάκληση χρησιμοποιώντας τον ``operator()`` στον γενικό functor που περάστηκε και στην συγκεκριμένη περίπτωση, παρέχει το ακέραιο (integer) όρισμα::
 
  void 
  LibraryFunction (Functor functor)
  {
    // Execute the library function
    functor(1234);
  }

.. Notice that ``LibraryFunction`` is completely decoupled from the specific type of the client.  The connection is made through the Functor polymorphism.
Παρατηρήστε ότι η ``LibraryFunction`` είναι πλήρως αποσυνδεδεμένη από τον συγκεκριμένο τύπι του πελάτη. Η σύνδεση πραγματοποιείται μέσω του Functor πολυμορφισμού.

.. The Callback API in |ns3| implements object-oriented callbacks using the functor mechanism.  This callback API, being based on C++ templates, is type-safe; that is, it performs static type checks to enforce proper signature compatibility between callers and callees.  It is therefore more type-safe to use than traditional function pointers, but the syntax may look imposing at first.  This section is designed to walk you through the Callback system so that you can be comfortable using it in |ns3|.
Η διεπαφη επικοινωνίας της επανάκλησης (Callback API) στον |ns3| υλοποιεί αντικειμενοστραφείς επανακλήσεις χρησιμοποιώντας τον μηχανισμό functor. Αυτή η API επανάκλησης, βασίζεται σε C++ πρότυπα (C++ templates), είναι type-safe, καθώς πραγματοποιείελέγχους στατικού τύπου προκειμένου να υποχρεώσουν σε συβατότητα υπογραφής μεταξύ αυτού που καλεί και αυτού που καλείται. Συνεπώς είναι περισσότερο type-safe να το χρησιμοποιησει κανείς, συγκριτικά με τους παραδοσιακούς δείκτες σε συνάρτηση, αλλά η σύντακη μπορεί αρχικά να αποθαρρύνει. Η συγκεκριμένη ενότητα έχει σχεδιαστεί ώστε να περιγράψει όλα τα χαρακτηριστικά του συτήματος επανακλήσεων ώστε να σας επιτρέψει να χρησιμοποιήσετε με άνεση τον |ns3|  


.. Using the Callback API
Χρησιμοποιώντας την API Επανακλήσεων
************************************

.. The Callback API is fairly minimal, providing only two services
Η διεπαφή επικοινωνίας επανακλήσεων έχει αντικειμενικά ελαχιστοποιημένη, παρέχοντας δύο υπηρεσίες:

.. 1. callback type declaration: a way to declare a type of callback with a given signature, and,
1. Την δήλωση τύπου επανάκλησης: έναν τρόπο να δηλώσεις τον τύπο της επανακλησης με μία δεδομέν υπογραφή, και

.. 2. callback instantiation: a way to instantiate a template-generated forwarding callback which can forward any calls to another C++ class member method or C++ function.
2. Δημιουργία στιγμιοτύπου επανάκλησης: ένας τρόπος να δημιουργήθεί ένας μηχανισμός προώθησης επανακλήσεων που μπορεί να προωθήσει κάθε κλήση σε κάποια μέθοδο μέλος μιας C++ κλάσης ή συνάτησης C++.

.. This is best observed via walking through an example, based on ``samples/main-callback.cc``.
Αυτό παρατηρείται καλύτερα μέσω ενός παραδείγματος, που βασίζεται στο ``samples/main-callback.cc``.

.. Using the Callback API with static functions
Χρησιμοποιώντας την API επανακλήσεων με στατικές συναρτήσεις
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. Consider a function
Θεωρήστε μία συνάρτηση ::

  static double
  CbOne (double a, double b)
  {
    std::cout << "invoke cbOne a=" << a << ", b=" << b << std::endl;
    return a;
  }

.. Consider also the following main program snippet
Θεωρήστε επίσης το παρακάτω απόσπασμα του κυρίως προγράμματος::

  int main (int argc, char *argv[])
  {
    // return type: double
    // first arg type: double
    // second arg type: double
    Callback<double, double, double> one;
  }

.. This is an example of a C-style callback -- one which does not include or need a ``this`` pointer.  The function template ``Callback`` is essentially the declaration of the variable containing the pointer-to-function.  In the example above, we explicitly showed a pointer to a function that returned an integer and took a single integer as a parameter,  The ``Callback`` template function is a generic version of that -- it is used to declare the type of a callback.  
Αυτό είναι ένα παράδειγμα επανάκλησης σε στυλ C -- το οποίο δεν περιλαμβάνει ή χρειάζεται ένα δείκτη ``this`` . Το πρότυπο της συνάρτησης ``Callback``  είναι ιδανικά η δήλωση της μεταβλητής που περιλαμβάνει τον δείκτη-σε-συνάρτηση. Στο παραπάνω παράδειγμα, δείξαμε ρητά έναν δείκτη σε συνάρτηση που επιστρέφει έναν ακέραιο και δέχεται έναν από ακέραιο σαν παράμετρο, το πρότυπο συνάρτηση ``Callback`` είναι μια γενική εκδοχή του προηγούμενου παραδείγματος -- χρησιμοποιείται για την δήλωση του τύπου της επανάκλησης.

.. Readers unfamiliar with C++ templates may consult `<http://www.cplusplus.com/doc/tutorial/templates/>`_.
.. note:: Οι αναγνώστες που δεν είναι εξοικειωμένοι με με τα πρότυπα της C++ μπορούν να συμβουλευτούν την ιστοσελίδα `<http://www.cplusplus.com/doc/tutorial/templates/>`_.

.. The ``Callback`` template requires one mandatory argument (the return type of the function to be assigned to this callback) and up to five optional arguments, which each specify the type of the arguments (if your particular callback function has more than five arguments, then this can be handled by extending the callback implementation).
Το πρότυπο ``Callback`` απαιτεί ένα υποχρεωτικό όρισμα (το τύπο επιστροφής της συνάρτησης που πρόκειται να ανατεθεί σε αυτή την επανάκληση) και μέχρι πέντε προαιρετικά ορίσματα, τα οποία μπορεί να καθορίζουν τον τύπο των ορισμάτων (εάν εάν η συνάρτηση επανακλησης που εξετάζουμε έχει περισσότερα από πέντε ορίσματα, τότε αυτό μπορεί να αντιμετωπιστεί με την επέκταση της υλοποίησης της επανακλησης). 

.. So in the above example, we have a declared a callback named "one" that will eventually hold a function pointer.  The signature of the function that it will hold must return double and must support two double arguments.  If one tries to pass a function whose signature does not match the declared callback, a compilation error will occur.  Also, if one tries to assign to a callback an incompatible one, compilation will succeed but a run-time  NS_FATAL_ERROR will be raised.  The sample program ``src/core/examples/main-callback.cc`` demonstrates both of these error cases at the end of the ``main()`` program.
Έτσι στο παραπάνω παράδειγμα, έχουμε δηλώσει μία επανάκληση με το όνομα "one" που τελικά θα κρατά έναν δείκτη συνάρτησης. Η υπογραφή της συνάρτησης που θα κρατά πρέπει να επιστρέφει double και να υποστηρίζει δύο ορίσματα double. Εάν κάποιος προσπαθήσει να περάσει μία συνάρτηση της οποίας η υπογραφή δεν ταιριάζει με την δηλωμένη επανάκληση, θα συμβεί ένα σφάλμα μεταγλώττισης. Επίσης, εάν κάποιος προσπαθήσει να αναθέσει σε μία επανάκληση μία μη συμβατή, τότε δεν θα προκληθεί σφάλμα μεταγλώττισης αλλά θα προκληθεί NS_FATAL_ERROR κατά τον χρόνο εκτέλεσης. Το πρόγραμμα-παράδειγμα στο αρχείο ``src/core/examples/main-callback.cc`` παρουσιάζει στο τέλος του προγράμματος ``main()`` και τις δύο περιπτώσεις λαθών που προαναφέρθηκαν. 

.. Now, we need to tie together this callback instance and the actual target function (CbOne).  Notice above that CbOne has the same function signature types as the callback-- this is important.  We can pass in any such properly-typed function  to this callback.  Let's look at this more closely
Στην συνέχεια, θα ενώσουμε το στιγμιότυπο της επανάκλησης με την πραγματική συνάρτηση στόχο (CbOne). Παρατηρήστε παραπάνω ότι η CbOne έχει την ίδια υπογραφή συνάρτησης όπως και η επανάκληση-- αυτό είναι σημαντικό. Μπορούμε να περάσουμε σε αυτήν επανάκληση κάθε τέτοια συνάρτηση που διαθέτει κατάλληλο τύπο. Ας την εξετάσουμε με περισσότερη προσοχή::

  static   double CbOne (double a, double b) {}
             ^             ^         ^
             |             |         |
             |             |         | 
  Callback<double,       double,   double> one;

.. You can only bind a function to a callback if they have the matching signature. The first template argument is the return type, and the additional template arguments are the types of the arguments of the function signature.
Μπορείς να προσδέσεις μία συνάρτηση σε μία επανάκληση εάν έχουν υπογραφή που να ταιριάζει. Το πρώτο όρισμα του προτύπου είναι ο τύπος επιστροφής, και τα επιπρόσθετα ορίσματα του προτύπου είναι οι τύποι των ορισμάτων της υπογραφής της συνάρτησης. 

.. Now, let's bind our callback "one" to the function that matches its signature
Ας προσδέσουμε στην συνέχεια την επανάκληση "one" με την συνάρτηση που ταιριάζει στην υπογραφή της::

  .. build callback instance which points to cbOne function
  //Δημιουργήστε ένα στιγμιότυπο της επανάκλησης που δείχνει στην συνάρτηση cbOne
  one = MakeCallback (&CbOne);

.. This call to ``MakeCallback`` is, in essence, creating one of the specialized functors mentioned above.  The variable declared using the ``Callback`` template function is going to be playing the part of the generic functor.  The assignment ``one = MakeCallback (&CbOne)`` is the cast that converts the specialized functor known to the callee to a generic functor known to the caller.
Αυτή η κληση στο  ``MakeCallback`` στην ουσία δημιουργεί ένα από τους εξειδικευμένους functors που αναφέραμε παραπάνω. Η μεταβλητή που δηλώθηκε να χρησιμοποιεί το πρότυπο συνάρτησης ``Callback`` πρόκειται να παίξει τον ρόλο του γενικού functor. H ανάθεση ``one = MakeCallback (&CbOne)`` είναι η μετατροπή του εξειδικευμένου functor, που είναι γνωστό στον καλούμενο σε γενικό functor που είναι γνωστό στον καλούντα.

.. Then, later in the program, if the callback is needed, it can be used as follows
Στην συνέχεια του προγράμματος, εάν χρειαστεί η επανάκληση, μπορεί να χρησιμοποιηθεί ως εξής::

  NS_ASSERT (!one.IsNull ());

  .. invoke cbOne function through callback instance
  //κάλεσε την συνάρτηση cbOne, μέσω του στιγμιοτύπου της επανάκλησης.
  double retOne;
  retOne = one (10.0, 20.0);

.. The check for ``IsNull()`` ensures that the callback is not null -- that there is a function to call behind this callback.  Then, ``one()`` executes the generic ``operator()`` which is really overloaded with a specific implementation of ``operator()`` and returns the same result as if ``CbOne()`` had been called directly.
Ο έλεγχος r ``IsNull()`` εξασφαλίζει ότι η επανάκληση δεν είναι null -- ότι υπάρχει μία συνάρτηση για να κληθεί πίσω από την επανάκληση. Στην συνέχεια η  ``one()`` εκτελεί τον γενικό ``operator()`` ο οποίος είναι υπερφορτωμένος με μία συγκεκριμένη υλοποίηση του ``operator()`` και επιστρέφει το ίδιο αποτέλεσμα σαν να είχε κληθεί απ ευθείας η ``CbOne()`` 

.. Using the Callback API with member functions
Χρησιμοποιώντας την API επανακλήσεων με συναρτήσεις μέλη (κλάσεων)
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. Generally, you will not be calling static functions but instead public member functions of an object.  In this case, an extra argument is needed to the MakeCallback function, to tell the system on which object the function should be invoked.  Consider this example, also from main-callback.cc
Γενικά, δεν θα καλείτε στατικές συναρτήσεις αλλά κυρίως δημόσιες συναρτήσεις ενός αντικειμένου. Σε αυτή την περίπτωση, ένα επιπλέον όρισμα απαιτείται από την συνάρτηση MakeCallback, ώστε να ενημερώσει το σύστημα ποιο αντικείμενο πρέπει να κληθεί. Σκεφτείτε το αυτό το παράδειγμα, επίσης από το main-callback.cc::


  class MyCb {
  public:
    int CbTwo (double a) {
        std::cout << "invoke cbTwo a=" << a << std::endl;
        return -5;
    }
  };

  int main ()
  {
    ...
    // return type: int
    // first arg type: double
    Callback<int, double> two;
    MyCb cb;
    // build callback instance which points to MyCb::cbTwo
    two = MakeCallback (&MyCb::CbTwo, &cb);
    ...
  }

.. Here, we pass an additional object pointer to the ``MakeCallback<>`` function. Recall from the background section above that ``Operator()`` will use the pointer to member syntax when it executes on an object
Εδώ, περνάμε ένα επιπλέον δείκτη στην συνάρτηση  ``MakeCallback<>`` . Θυμηθείτε από την ενοτητα παρασκήνιο (bacground) ότι ο  ``Operator()`` θα χρησιμοποιήσει ένα δείκτη σε μέλος, όταν θα εκτελεστεί σε ένα αντικείμενο::

      virtual int operator() (ARG arg)
      {
        (*m_p.*m_pmi)(arg);
      }

.. And so we needed to provide the two variables (``m_p`` and ``m_pmi``) when we made the specific functor.  The line
Και έτσι χρειάζεται να παρέχουμε τις δυο μεταβλητές (``m_p`` and ``m_pmi``) όταν δημιουργήσουμε τον συγκεκριμένο functor. Η γραμμή::

    two = MakeCallback (&MyCb::CbTwo, &cb);

.. does precisely that.  In this case, when ``two ()`` is invoked
πραγματοποιεί ακριβώς αυτό, Σε αυτή την περίπτωση, όταν καλείται η ``two ()`` :: 

  int result = two (1.0);

.. will result in a call tothe ``CbTwo`` member function (method) on the object pointed to by ``&cb``.  
θα έχει ως αποτέλεσμα την κλήση της συνάρτησης μέλους ``CbTwo`` του αντικειμένου που δείχνεται από τον ``&cb``.

.. Building Null Callbacks
Δημιουργώντας Null Επανακλήσεις
+++++++++++++++++++++++++++++++

It is possible for callbacks to be null; hence it may be wise to
check before using them.  There is a special construct for a null
callback, which is preferable to simply passing "0" as an argument;
it is the ``MakeNullCallback<>`` construct::

  two = MakeNullCallback<int, double> ();
  NS_ASSERT (two.IsNull ());

Invoking a null callback is just like invoking a null function pointer: it will
crash at runtime.

Bound Callbacks
***************

A very useful extension to the functor concept is that of a Bound Callback.  
Previously it was mentioned that closures were originally function calls 
packaged up for later execution.  Notice that in all of the Callback 
descriptions above, there is no way to package up any parameters for use 
later -- when the ``Callback`` is called via ``operator()``.  All of 
the parameters are provided by the calling function.  

What if it is desired to allow the client function (the one that provides the
callback) to provide some of the parameters?  `Alexandrescu <http://erdani.com/book/main.html>`_ calls the process of
allowing a client to specify one of the parameters *"binding"*.  One of the 
parameters of ``operator()`` has been bound (fixed) by the client.

Some of our pcap tracing code provides a nice example of this.  There is a
function that needs to be called whenever a packet is received.  This function
calls an object that actually writes the packet to disk in the pcap file 
format.  The signature of one of these functions will be::

  static void DefaultSink (Ptr<PcapFileWrapper> file, Ptr<const Packet> p);

The static keyword means this is a static function which does not need a
``this`` pointer, so it will be using C-style callbacks.  We don't want the
calling code to have to know about anything but the Packet.  What we want in
the calling code is just a call that looks like::

  m_promiscSnifferTrace (m_currentPkt);

What we want to do is to *bind* the ``Ptr<PcapFileWriter> file`` to the 
specific callback implementation when it is created and arrange for the 
``operator()`` of the Callback to provide that parameter for free.

We provide the ``MakeBoundCallback`` template function for that purpose.  It
takes the same parameters as the ``MakeCallback`` template function but also
takes the parameters to be bound.  In the case of the example above::

    MakeBoundCallback (&DefaultSink, file);

will create a specific callback implementation that knows to add in the extra
bound arguments.  Conceptually, it extends the specific functor described above
with one or more bound arguments::

  template <typename T, typename ARG, typename BOUND_ARG>
  class SpecificFunctor : public Functor
   {
   public:
      SpecificFunctor(T* p, int (T::*_pmi)(ARG arg), BOUND_ARG boundArg)
      {
        m_p = p;
        m_pmi = pmi;
        m_boundArg = boundArg;
      }

      virtual int operator() (ARG arg)
      {
        (*m_p.*m_pmi)(m_boundArg, arg);
      }
  private:
      void (T::*m_pmi)(ARG arg);
      T* m_p;
      BOUND_ARG m_boundArg;
   };

You can see that when the specific functor is created, the bound argument is saved
in the functor / callback object itself.  When the ``operator()`` is invoked with
the single parameter, as in::

  m_promiscSnifferTrace (m_currentPkt);

the implementation of ``operator()`` adds the bound parameter into the actual
function call::

  (*m_p.*m_pmi)(m_boundArg, arg);

It's possible to bind two or three arguments as well.  Say we have a function with
signature::

  static void NotifyEvent (Ptr<A> a, Ptr<B> b, MyEventType e);

One can create bound callback binding first two arguments like::

  MakeBoundCallback (&NotifyEvent, a1, b1);

assuming `a1` and `b1` are objects of type `A` and `B` respectively.  Similarly for
three arguments one would have function with a signature::

  static void NotifyEvent (Ptr<A> a, Ptr<B> b, MyEventType e);

Binding three arguments in done with::

  MakeBoundCallback (&NotifyEvent, a1, b1, c1);

again assuming `a1`, `b1` and `c1` are objects of type `A`, `B` and `C` respectively.

This kind of binding can be used for exchanging information between objects in
simulation; specifically, bound callbacks can be used as traced callbacks, which will
be described in the next section.

Traced Callbacks
****************
*Placeholder subsection*

Callback locations in ns-3
**************************

Where are callbacks frequently used in |ns3|?  Here are some of the
more visible ones to typical users:

* Socket API
* Layer-2/Layer-3 API
* Tracing subsystem
* API between IP and routing subsystems

Implementation details
**********************

The code snippets above are simplistic and only designed to illustrate the mechanism
itself.  The actual Callback code is quite complicated and very template-intense and
a deep understanding of the code is not required.  If interested, expert users may
find the following useful.

The code was originally written based on the techniques described in 
`<http://www.codeproject.com/cpp/TTLFunction.asp>`_.
It was subsequently rewritten to follow the architecture outlined in 
`Modern C++ Design, Generic Programming and Design Patterns Applied, Alexandrescu, chapter 5, Generalized Functors <http://www.moderncppdesign.com/book/main.html>`_.

This code uses:

* default template parameters to saves users from having to
  specify empty parameters when the number of parameters
  is smaller than the maximum supported number
* the pimpl idiom: the Callback class is passed around by
  value and delegates the crux of the work to its pimpl pointer.
* two pimpl implementations which derive from CallbackImpl
  FunctorCallbackImpl can be used with any functor-type
  while MemPtrCallbackImpl can be used with pointers to
  member functions.
* a reference list implementation to implement the Callback's
  value semantics.

This code most notably departs from the Alexandrescu implementation in that it
does not use type lists to specify and pass around the types of the callback 
arguments. Of course, it also does not use copy-destruction semantics and 
relies on a reference list rather than autoPtr to hold the pointer.
