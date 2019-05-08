# doc
```
#include <stdio.h>
#include <stdint.h>
int showbits (unsigned int n)
{
		unsigned int i = 32;
	    uint8_t dizi[32]={0}, a=0,d=0;

	for (; i > 0; i--)
	{
		  if(	n & (1 << (i - 1)))
	        {
	        //    printf ("1");
	            dizi[a++]=i;
	            
	        }
	        else
	        {
	      //      printf ("0");
	        }
    }
	       // printf("dizi b: %d 0: %d 1: %d ",(int)(sizeof(dizi)-sizeof(dizi[0])),dizi[0],dizi[1]);
	        for(int i=0;i<(sizeof(dizi)-sizeof(dizi[0]));i++)
	        {
                if(dizi[i+1]!=0)
                {
    	        uint8_t cSifir=(dizi[i]-dizi[i+1])-1;
    	        if(cSifir>=d)
    	            d=cSifir;
    	        }
    	        else 
    	        {
    	           // break;
    	        }
	        }
	      //  printf("cevap: %d",d);
	      return d;
}

int main()
{
      //  printf("d: %d",);
    return showbits(1041);
}

```

2-)*************************************************************************************************************************************
```
// you can write to stdout for debugging purposes, e.g.
// printf("this is a debug message\n");
#include <stdio.h>
#include <string.h>

struct Results solution(int A[], int N, int K) {
    struct Results result;
    // write your code in C99 (gcc 6.2.0)
    result.A = A;
    result.N = N;
    for(int a=0;a<K;a++)
    {
  /*  printf("RES A: ");
   for(int i=0; i<N;i++)
        printf(" %d ",result.A[i]);
    printf("\n");*/
   
    int f[N];
    //printf("a: %d",*(result.A+(N-1)));
    f[0]=*(result.A+(N-1));
    for(int i=0;i<N-1;i++)
    {
    *(f+(i+1))=A[i];
    }
  /*  printf("F: ");
   for(int i=0; i<N;i++)
    printf(" %d ",f[i]);
    printf("\n");*/
   // result.A=f;
    memset(result.A, 0, sizeof( f) );
    memcpy(result.A, f,sizeof( f));

    }
     return result;
}
```

3-)********************************************************************************************************************************************************
```

// you can write to stdout for debugging purposes, e.g.
// printf("this is a debug message\n");
#include <stdio.h>
int solution(int A[], int N) {
    // write your code in C99 (gcc 6.2.0)
     int  missing_int = 0;
    for(int i=0; i<N;i++)
    {
        missing_int^=A[i];
        
    }
    return missing_int;
}
```

