## 2019年1月30日 HZ 晴朗  风暴-0130


Read code 

#### Lock-free “annihilating” dual stack
```c

struct cptr {                // counted pointer
    snode *ptr;
    int sn;
};  // 64-bit datatype

struct tptr {                // tagged pointer
    snode *30 ptr;
    bool is_request;         // tags describe
    bool data_underneath;    // pointed-to node
};  // 32-bit datatype

struct ctptr extends tptr {  // counted tagged pointer
    int sn;
};  // 64-bit datatype

struct dualstack {
    ctptr head;
};

struct snode {               // stack node
    union {
        int data;
        cptr data_node;      // data must overlie ptr, not sn
    };
    tptr next;
};

void ds_init(dualstack *S)
{
    stack->head.ptr = NULL;
}

void push(int v, dualstack *S)
{
    snode *n = new snode;
    n->data = v;

    while (1) {
        ctptr head = S->head;
        n->next = head;
        if (head.ptr == NULL || (!head.is_request && !head.data_underneath)) {
            if (cas(&S->head, head, {{n, FALSE, FALSE}, head.sn+1})) return;
        } else if (head.is_request) {
            tptr next = head.ptr->next;
            cptr old = head.ptr->data_node;
            // link in filler node
            if (!cas(&S->head, head, {{n, FALSE, TRUE}, head.sn+1}))
                continue;   // someone else fulfilled the request
            // fulfill request node
            (void) cas(&head.ptr->data_node, old, {n, old.sn+1});
            // link out filler and request
            (void) cas(&S->head, {{n, FALSE, TRUE}, head.sn+1}, {next, head.sn+2});
            return;
        } else {    // data underneath; need to help
            tptr next = head.ptr->next;
            if (next.ptr == NULL) continue;  // inconsistent snapshot
            cptr old = next.ptr->data_node;
            if (head != S->head) continue;   // inconsistent snapshot
            // fulfill request node
            if (old.ptr == NULL)
                (void) cas(&next.ptr->data_node, old, {head.ptr, old.sn+1});
            // link out filler and request
            (void) cas(&S->head, head, {next->next, head.sn+1});
        }
    }
}

int pop(dualstack *S{, thread_id r})
{
    snode *n = NULL;

    while (1) {
        ctptr head = S->head;
        if (!head.is_request && !head.data_underneath) {
            tptr next = head.ptr->next;
            if (cas(&S->head, head, {next, head.sn+1})) {
                int result = head.ptr->data;
                delete head.ptr;
                if (n != NULL) delete n;
                return result;
            }
        } else if (head.ptr == NULL || head.is_request) {
            if (n == NULL) {
                n = new snode;
                n->data_node.ptr = NULL;
            }
            n->next = {head.ptr, TRUE, FALSE};
            if (!cas(&S->head, head, {{n, TRUE, FALSE}, head.sn+1}))
                continue;   // couldn't push request

            // initial linearization point

            while (n->data_node.ptr == NULL); // local spin
            // help remove my request node if needed
            head = S->head;
            if (head.ptr == n)
                (void) cas(&S->head, head, {n->next, head.sn+1});
            int result = n->data_node.ptr->data;
            delete n->data_node.ptr;  delete n;
            return result;
        } else {    // data underneath; need to help
            tptr next = head.ptr->next;
            if (next.ptr == NULL) continue;  // inconsistent snapshot
            cptr old = next.ptr->data_node;
            if (head != S->head) continue;   // inconsistent snapshot
            // fulfill request node
            if (old.ptr == NULL)
                (void) cas(&next.ptr->data_node, old, {head.ptr, old.sn+1});
            // link out filler and request
            (void) cas(&S->head, head, {next->next, head.sn+1});
        }
    }
}

```

#### Lock-free dualqueue

```c
struct cptr {                // counted pointer
    qnode *ptr;
    int sn;
};  // 64-bit datatype

struct ctptr {              // counted tagged pointer
    qnode *31 ptr;
    bool is_request;        // tag describes pointed-to node
    int sn;
};  // 64-bit datatype

struct qnode {
    cval data;
    cptr request;
    ctptr next;
};

struct dualqueue {
    cptr head;
    ctptr tail;
};

void dq_init(dualqueue *Q)
{
    qnode *qn = new qnode;
    qn->next.ptr = NULL;
    Q->head.ptr = Q->tail.ptr = qn;
    Q->tail.is_request = FALSE;
}

void enqueue(int v, dualqueue *Q)
{
    qnode *n = new qnode;
    n->data = v;
    n->next.ptr = n->request.ptr = NULL;
    while (1) {
        ctptr tail = Q->tail;
        cptr head = Q->head;
        if (tail.ptr == head.ptr) || !tail.is_request) {
            // queue empty, tail falling behind, or queue contains data (queue could also 
            // contain exactly one outstanding request with tail pointer as yet unswung)
            cptr next = tail.ptr->next;
            if (tail == Q->tail) {  // tail and next are consistent
                if (next.ptr != NULL) {     // tail falling behind
                    (void) cas(&Q->tail, tail, {{next.ptr, next.is_request}, tail.sn+1});
                } else {    // try to link in the new node
                    if (cas(&tail.ptr->next, next, {{n, FALSE}, next.sn+1})) {
                        (void) cas(&Q->tail, tail, {{n, FALSE}, tail.sn+1});
                        return;
                    }
                }
            }
        } else {    // queue consists of requests
            ctptr next = head.ptr->next;
            if (tail == Q->tail) {      // tail has not changed
                cptr req = head.ptr->request;
                if (head == Q->head) {  // head, next, and req are consistent
                    bool success = (req.ptr == NULL
                        && cas(&head.ptr->request, req, {n, req.sn+1}));
                    // try to remove fulfilled request even if it's not mine
                    (void) cas(&Q->head, head, {next.ptr, head.sn+1});
                    if (success) return;
                }
            }
        }
    }
}

int dequeue(dualqueue *Q{, thread_id r})
{
    qnode *n = new qnode;
    n->is_request = TRUE;
    n->ptr = n->request = NULL;

    while (1) {
        cptr head  = Q->head;
        ctptr tail = Q->tail;
        if ((tail.ptr == head.ptr) || tail.is_request) {
            // queue empty, tail falling behind, or queue contains data (queue could also 
            // contain exactly one outstanding request with tail pointer as yet unswung)
            cptr next = tail.ptr->next;
            if (tail == Q->tail) {  // tail and next are consistent
                if (next.ptr != NULL) {     // tail falling behind
                    (void) cas(&Q->tail, tail, {{next.ptr, next.is_request}, tail.sn+1});
                } else {    // try to link in a request for data
                    if (cas(&tail.ptr->next, next, {{n, TRUE}, next.sn+1})) {
                        // linked in request; now try to swing tail pointer
                        (void) cas(&Q->tail, tail, {{n, TRUE}, tail.sn+1}) {
                        // help someone else if I need to
                        if (head == Q->head && head.ptr->request.ptr != NULL) {
                            (void)cas(&Q->head, head, {head.ptr->next.ptr, head.sn+1});
                        }

                        // initial linearization point

                        while (tail.ptr->request.ptr == NULL);  // spin
                        // help snip my node
                        head =  Q->head;
                        if (head.ptr == tail.ptr) {
                            (void) cas(&Q->head, head, {n, head.sn+1});
                        }
                        // data is now available; read it out and go home
                        int result = tail.ptr->request.ptr->data;
                        delete tail.ptr->request.ptr;  delete tail.ptr;
                        return result;
                    }
                }
            }
        } else {    // queue consists of real data
            cptr next = head.ptr->next;
            if (tail == Q->tail) {
                // head and next are consistent; read result *before* swinging head
                int result = next.ptr->data;
                if (cas(&Q->head, head, {next.ptr, head.sn+1})) {
                    delete head.ptr;  delete n;
                    return result;
                }
            }
        }
    }
}
```

