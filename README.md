# Starve_free_reader-writer_problem
Assumptions:Semaphores uses a FIFO queue to maintain the list of blocked processes.
Creating the FIFO queue to maintain the list of blocked processes:
``` cpp
struct Semaphore{
  int value=1;
  FIFO_Queue* K=new FIFO_Queue();
}
    
void wait(Semaphore *S,int* process_id){
  S->value--;
  if(S->value<0){
  S->K->push(process_id);
  block(); /*This function will block the process by using system call and will transfer it to the waiting queue.
           The process will remain in the waiting queue till it is waken up by the wakeup() system calls.
           This is a type of non busy waiting.*/
  }}
    
void signal(Semaphore *S){
  S->value=S->value+1;
  if(S->value<1){
  int* p_id =S->K->pop();
  wakeup(p_id); //This function will wakeup the process with the given p_id using system calls.
  }}

 // A Process Block.
struct ProcessBlock{
    ProcessBlock* next;
    int* process_block;
  }

struct FIFO_Queue{
    ProcessBlock* front, rear;
    int* pop(){
        if(front==NULL){return -1;}// Error : underflow.
        else{
            int* val=front->value;
            front=front->next;
            if(front==NULL){
                rear=NULL;}
            return val;
              }  }
    void* push(int* val){
        ProcessBlock* b=new ProcessBlock();
        b->value=val;
        if(rear==NULL){
            front=n;rear=n;}          
        else{
            rear->next=b;rear=b;}
                        }}
                        ```
    
Creating the semaphores:

int reader_count=0;       //Integer representing the number of reader executing critical section.
Semaphore turn=new Semaphore();        /*seamphore representing the order in which the writer and 
                                         reader are requesting the access to critical section.*/
Semaphore cs=new Semaphore();        //seamphore required to access the critical section.
Semaphore mutex=new Semaphore();       //seamphore required to change the reader_count variable.


Reader's Code:
do{
                  /*  ENTRY SECTION   */

       wait(turn,process_id);          //waiting for its turn to get executed.
       wait(mutex,process_id);           //requesting access to change reader_count.
       reader_count++;             //update the number of readers trying to access critical section .
       if(reader_count==1)                 //If i'm the first reader,then request access to critical section.
         {wait(cs);}          //requesting  access to the critical section for readers.
       signal(turn);             //releasing turn so that the next reader or writer can take the token and can be serviced.
       signal(mutex);           //release access to the reader_count.

                  /* CRITICAL SECTION */
                  /*   EXIT SECTION   */

       wait(mutex,process_id)         //requesting access to change reader_count         
       reader_count--;         //A reader has finished executing critical section and so the reader_count should be decrease by 1.
       if(reader_count==0)                //If all the reader have finished executing their critical section.
        {signal(cs);}               //releasing access to critical section for next reader or writer.
       signal(mutex);                    //release access to the reader_count.  
                      
}while(true);



Writer's code:
do{
                  /*  ENTRY SECTION  */

      wait(turn,process_id);             //waiting for its turn to get executed.
      wait(cs,process_id);             //requesting  access to the critical section.
      signal(turn,process_id);            //releasing turn so that the next reader or writer can take the token and can be serviced.
                                          
                 /* CRITICAL SECTION */
                 /*   EXIT SECTION   */

      signal(cs);                         //releasing access to critical section for next reader or writer

}while(true);
