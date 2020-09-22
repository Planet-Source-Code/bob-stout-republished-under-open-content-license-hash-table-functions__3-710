<div align="center">

## Hash table functions


</div>

### Description

Hash table management by Jerry Coffin, with improvements by HenkJan Wolthuis.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Bob Stout \(republished under Open Content License\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/bob-stout-republished-under-open-content-license.md)
**Level**          |Intermediate
**User Rating**    |4.5 (18 globes from 4 users)
**Compatibility**  |C, C\+\+ \(general\), Microsoft Visual C\+\+, Borland C\+\+, UNIX C\+\+
**Category**       |[Data Structures](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/data-structures__3-8.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/bob-stout-republished-under-open-content-license-hash-table-functions__3-710/archive/master.zip)

### API Declarations

```
/* +++Date last modified: 05-Jul-1997 */
#ifndef HASH__H
#define HASH__H
#include <stddef.h>      /* For size_t   */
/*
** A hash table consists of an array of these buckets. Each bucket
** holds a copy of the key, a pointer to the data associated with the
** key, and a pointer to the next bucket that collided with this one,
** if there was one.
*/
typedef struct bucket {
  char *key;
  void *data;
  struct bucket *next;
} bucket;
/*
** This is what you actually declare an instance of to create a table.
** You then call 'construct_table' with the address of this structure,
** and a guess at the size of the table. Note that more nodes than this
** can be inserted in the table, but performance degrades as this
** happens. Performance should still be quite adequate until 2 or 3
** times as many nodes have been inserted as the table was created with.
*/
typedef struct hash_table {
  size_t size;
  bucket **table;
} hash_table;
/*
** This is used to construct the table. If it doesn't succeed, it sets
** the table's size to 0, and the pointer to the table to NULL.
*/
hash_table *construct_table(hash_table *table,size_t size);
/*
** Inserts a pointer to 'data' in the table, with a copy of 'key' as its
** key. Note that this makes a copy of the key, but NOT of the
** associated data.
*/
void *insert(char *key,void *data,struct hash_table *table);
/*
** Returns a pointer to the data associated with a key. If the key has
** not been inserted in the table, returns NULL.
*/
void *lookup(char *key,struct hash_table *table);
/*
** Deletes an entry from the table. Returns a pointer to the data that
** was associated with the key so the calling code can dispose of it
** properly.
*/
void *del(char *key,struct hash_table *table);
/*
** Goes through a hash table and calls the function passed to it
** for each node that has been inserted. The function is passed
** a pointer to the key, and a pointer to the data associated
** with it.
*/
void enumerate(struct hash_table *table,void (*func)(char *,void *));
/*
** Frees a hash table. For each node that was inserted in the table,
** it calls the function whose address it was passed, with a pointer
** to the data that was in the table. The function is expected to
** free the data. Typical usage would be:
** free_table(&table, free);
** if the data placed in the table was dynamically allocated, or:
** free_table(&table, NULL);
** if not. ( If the parameter passed is NULL, it knows not to call
** any function with the data. )
*/
void free_table(hash_table *table, void (*func)(void *));
#endif /* HASH__H */
```


### Source Code

```
#include <stdlib.h>
#include "hash.h"
/*
** public domain code by Jerry Coffin, with improvements by HenkJan Wolthuis.
**
** Tested with Visual C 1.0 and Borland C 3.1.
** Compiles without warnings, and seems like it should be pretty
** portable.
*/
/*
** These are used in freeing a table. Perhaps I should code up
** something a little less grungy, but it works, so what the heck.
*/
static void (*function)(void *) = (void (*)(void *))NULL;
static hash_table *the_table = NULL;
/* Initialize the hash_table to the size asked for. Allocates space
** for the correct number of pointers and sets them to NULL. If it
** can't allocate sufficient memory, signals error by setting the size
** of the table to 0.
*/
hash_table *construct_table(hash_table *table, size_t size)
{
   size_t i;
   bucket **temp;
   table -> size = size;
   table -> table = (bucket * *)malloc(sizeof(bucket *) * size);
   temp = table -> table;
   if ( temp == NULL )
   {
      table -> size = 0;
      return table;
   }
   for (i=0;i<size;i++)
      temp[i] = NULL;
   return table;
}
/*
** Hashes a string to produce an unsigned short, which should be
** sufficient for most purposes.
*/
static unsigned hash(char *string)
{
   unsigned ret_val = 0;
   int i;
   while (*string)
   {
      i = *( int *)string;
      ret_val ^= i;
      ret_val <<= 1;
      string ++;
   }
   return ret_val;
}
/*
** Insert 'key' into hash table.
** Returns pointer to old data associated with the key, if any, or
** NULL if the key wasn't in the table previously.
*/
void *insert(char *key, void *data, hash_table *table)
{
   unsigned val = hash(key) % table->size;
   bucket *ptr;
   /*
   ** NULL means this bucket hasn't been used yet. We'll simply
   ** allocate space for our new bucket and put our data there, with
   ** the table pointing at it.
   */
   if (NULL == (table->table)[val])
   {
      (table->table)[val] = (bucket *)malloc(sizeof(bucket));
      if (NULL==(table->table)[val])
         return NULL;
      (table->table)[val] -> key = strdup(key);
      (table->table)[val] -> next = NULL;
      (table->table)[val] -> data = data;
      return (table->table)[val] -> data;
   }
   /*
   ** This spot in the table is already in use. See if the current string
   ** has already been inserted, and if so, increment its count.
   */
   for (ptr = (table->table)[val];NULL != ptr; ptr = ptr -> next)
      if (0 == strcmp(key, ptr->key))
      {
         void *old_data;
         old_data = ptr->data;
         ptr -> data = data;
         return old_data;
      }
   /*
   ** This key must not be in the table yet. We'll add it to the head of
   ** the list at this spot in the hash table. Speed would be
   ** slightly improved if the list was kept sorted instead. In this case,
   ** this code would be moved into the loop above, and the insertion would
   ** take place as soon as it was determined that the present key in the
   ** list was larger than this one.
   */
   ptr = (bucket *)malloc(sizeof(bucket));
   if (NULL==ptr)
      return 0;
   ptr -> key = strdup(key);
   ptr -> data = data;
   ptr -> next = (table->table)[val];
   (table->table)[val] = ptr;
   return data;
}
/*
** Look up a key and return the associated data. Returns NULL if
** the key is not in the table.
*/
void *lookup(char *key, hash_table *table)
{
   unsigned val = hash(key) % table->size;
   bucket *ptr;
   if (NULL == (table->table)[val])
      return NULL;
   for ( ptr = (table->table)[val];NULL != ptr; ptr = ptr->next )
   {
      if (0 == strcmp(key, ptr -> key ) )
         return ptr->data;
   }
   return NULL;
}
/*
** Delete a key from the hash table and return associated
** data, or NULL if not present.
*/
void *del(char *key, hash_table *table)
{
   unsigned val = hash(key) % table->size;
   void *data;
   bucket *ptr, *last = NULL;
   if (NULL == (table->table)[val])
      return NULL;
   /*
   ** Traverse the list, keeping track of the previous node in the list.
   ** When we find the node to delete, we set the previous node's next
   ** pointer to point to the node after ourself instead. We then delete
   ** the key from the present node, and return a pointer to the data it
   ** contains.
   */
   for (last = NULL, ptr = (table->table)[val];
      NULL != ptr;
      last = ptr, ptr = ptr->next)
   {
      if (0 == strcmp(key, ptr -> key))
      {
         if (last != NULL )
         {
            data = ptr -> data;
            last -> next = ptr -> next;
            free(ptr->key);
            free(ptr);
            return data;
         }
         /*
         ** If 'last' still equals NULL, it means that we need to
         ** delete the first node in the list. This simply consists
         ** of putting our own 'next' pointer in the array holding
         ** the head of the list. We then dispose of the current
         ** node as above.
         */
         else
         {
            data = ptr->data;
            (table->table)[val] = ptr->next;
            free(ptr->key);
            free(ptr);
            return data;
         }
      }
   }
   /*
   ** If we get here, it means we didn't find the item in the table.
   ** Signal this by returning NULL.
   */
   return NULL;
}
/*
** free_table iterates the table, calling this repeatedly to free
** each individual node. This, in turn, calls one or two other
** functions - one to free the storage used for the key, the other
** passes a pointer to the data back to a function defined by the user,
** process the data as needed.
*/
static void free_node(char *key, void *data)
{
   (void) data;
   if (function)
      function(del(key,the_table));
   else del(key,the_table);
}
/*
** Frees a complete table by iterating over it and freeing each node.
** the second parameter is the address of a function it will call with a
** pointer to the data associated with each node. This function is
** responsible for freeing the data, or doing whatever is needed with
** it.
*/
void free_table(hash_table *table, void (*func)(void *))
{
   function = func;
   the_table = table;
   enumerate( table, free_node);
   free(table->table);
   table->table = NULL;
   table->size = 0;
   the_table = NULL;
   function = (void (*)(void *))NULL;
}
/*
** Simply invokes the function given as the second parameter for each
** node in the table, passing it the key and the associated data.
*/
void enumerate( hash_table *table, void (*func)(char *, void *))
{
   unsigned i;
   bucket *temp;
   for (i=0;i<table->size; i++)
   {
      if ((table->table)[i] != NULL)
      {
         for (temp = (table->table)[i];
            NULL != temp;
            temp = temp -> next)
         {
            func(temp -> key, temp->data);
         }
      }
   }
}
#ifdef TEST
#include <stdio.h>
void printer(char *string, void *data)
{
   printf("%s: %s\n", string, (char *)data);
}
int main(void)
{
   hash_table table;
   char *strings[] = {
      "The first string",
      "The second string",
      "The third string",
      "The fourth string",
      "A much longer string than the rest in this example.",
      "The last string",
      NULL
      };
   char *junk[] = {
      "The first data",
      "The second data",
      "The third data",
      "The fourth data",
      "The fifth datum",
      "The sixth piece of data"
      };
   int i;
   void *j;
   construct_table(&table,200);
   for (i = 0; NULL != strings[i]; i++ )
      insert(strings[i], junk[i], &table);
   for (i=0;NULL != strings[i];i++)
   {
      printf("\n");
      enumerate(&table, printer);
      del(strings[i],&table);
   }
   for (i=0;NULL != strings[i];i++)
   {
      j = lookup(strings[i], &table);
      if (NULL == j)
         printf("\n'%s' is not in table",strings[i]);
      else printf("\nERROR: %s was deleted but is still in table.",
         strings[i]);
   }
   free_table(&table, NULL);
   return 0;
}
#endif /* TEST */
```

