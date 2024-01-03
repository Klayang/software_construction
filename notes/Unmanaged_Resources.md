###### Managed Resources

- Managed resources mean *managed memory* managed by the garbage collector

- When you no longer have reference to an object (which uses managed memory)
  
  - The garbage collector will (eventually) release that memory for you

    

###### Unmanaged Resources

Unmanaged resources are everything that garbage collector doesn't know about

- e.g., open files, open network connections, unmanaged memory, etc

- Normally you want to release those unmanaged resources before you lose all ref
  
  - By calling `Dispose` on the object, or wrapping it around `using` statement

- If you forgot to dispose unmanaged resources, the garbage collector'd handle it
  
  - When the object containing that resource is garbage-collected

- But since the garbage collector doesn't know the unmanaged resources:
  
  - It can't tell how badly it needs to release them, so it's possible
  
  - That your program'd perform poorly or run out of resources entirely

    
