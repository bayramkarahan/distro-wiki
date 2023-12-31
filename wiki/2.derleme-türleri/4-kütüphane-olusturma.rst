Kütüphane oluşturma
^^^^^^^^^^^^^^^^^^^
Kütühane dosyaları **so** uzantılıdır ve ihtiyaç duyulan diğer yazılımlar tarafından kullanılır.
Kütüphane oluşturmak için öncelikle aşağıdaki gibi bir C kodumuz olsun.

.. code-block:: C

	//deneme.c dosyası
	#include <stdio.h>
	void yazdir(){
	    printf("Merhaba Dünya\n");
	}

Bu dosyayı doğrudan derlersek **main** fonksiyonu bulunmadığı için aşağıdaki gibi bir hata ile karşılaşırız.

.. code-block:: shell

	$ gcc deneme.c -o libdeneme.so 
	    ...: in function `_start':
	    (.text+0x17): undefined reference to main'
	    collect2: error: ld returned 1 exit status

Kütüphane derlemek için aşağıdaki iki komutu arka arkaya kullanmalıyız.

İlk satır **denem.o** dosyası oluşturacaktır. 
ikinci satırımızda **-shared** parametresi kullanarak **main** bulunmayan kütüphane dosyamız derlendi ve **deneme.so** dosyası oluşturulmuş olur.

.. code-block:: shell

	$ gcc -c -Wall -Werror -fpic deneme.c
	$ gcc -shared -o libdeneme.so deneme.o

Kütüphane aşağıdaki gibi kullanılabilir.

.. code-block:: C

	extern void yazdir();
	int main(){
	    yazdir();
	}

Şimdi bu kütüphaneyi başka bir kodu derlemek için kullanalım. Bunun için **-L** parametresi ile kütüphanenin bulunduğu yeri göstermeliyiz. **-l** parametresi ile de kütüphaneyi bağlamalıyız.

.. code-block:: shell

	$ gcc -L/ders/kutuphane -o main main.c -ldeneme

Kütüphaneyi Sisteme Dahil Etme ve Kullanma
++++++++++++++++++++++++++++++++++++++++++

Yukarıdaki örnekte /ders/kutuphane/libdeneme.so dosyasını kullandık.

Artık derlediğimiz main adındaki ikili dosyayı çalıştırabiliriz. Çalıştırdığımızda aşağıdaki gibi hatayla karşılaşırız.
libdeneme.so dosyasını bulamadığını söylüyor.

.. code-block:: shell

	$ ./main 
	./main: error while loading shared libraries: libdeneme.so: cannot open shared object file: No such file or directory

Bu hatayı daha net anlamak için ve **main** ikili dosyamızın hangi dosyalara ihtiyacı olduğunu görmek için aşağıdaki komutu çalıştırırız.
**main** ikili dosyasının çalışması için 4 tane dosyaya ihtiyacı var. Bunlardan birisi bizim oluşturduğumuz **libdeneme.so** dosyası.
Fakat dosyayı bulamadığını söylüyor. Aslında kütüphaneyi **/usr/lib/libdeneme.so** konumunda olup olmadığını göre bu mesajı veriyor.

.. code-block:: shell

	$ ldd ./main
	linux-vdso.so.1 (0x00007fffab5e8000)
	libdeneme.so => not found
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff73002c000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ff73020d000)

**libdeneme.so** dosyamızı **/usr/lib/** konumuna kopyalayalım. Erişim izni verelim. **ldd ./main** komutunu çalıştırdığımızda
artık dosyanın karşısında **not found** mesajı yok. Artık çalıcaşacaktır. 

.. code-block:: shell

	$ sudo cp /ders/kutuphane/libdeneme.so /usr/lib
	$ sudo chmod 0755 /usr/lib/libdeneme.so
	$ ldd ./main
	linux-vdso.so.1 (0x00007ffdf93fc000)
	libdeneme.so => /lib/libdeneme.so (0x00007fa5c281d000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fa5c2658000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fa5c283e000)
	$ ./main 
 	Merhaba Dünya

Kütüphane Dosyasının Konumunu İsteğe Göre Belirleme(rpath)
++++++++++++++++++++++++++++++++++++++++++++++++

Bazen kütüphane dosyalarının **/usr/lib** konumunda değilde bizim belirleyeceğimiz konumda olmasını isteyebiliriz.

* Örneğin **/opt/main/** konumunda olmasını istersek aşağıdaki gibi yapmalıyız. 
* Hatasız bir sonuç almak için öncelikle **/usr/lib/libdeneme.so** konumundaki dosyamızı silelim. 
* Daha sonra **/opt/main/** konumunda olacak şekilde main ikili dosyamızı derleyelim. 
* Eğer **/opt/main** klasörü yoksa oluşturmalıyız. Ben olmadığını varsayıyorum ve oluşturuyorum.
* **libdeneme.so** dosyamızıda **/usr/lib/libdeneme.so** konumuna kopyalayıp izinlerini ayarlayalım. 
* Son işlem olarak test edelim.

Bunun için;

.. code-block:: shell
	
	$ sudo rm /usr/lib/libdeneme.so
	$ gcc -L/ders/kutuphane -Wl,-rpath=/opt/main -Wall -o main main.c -ldeneme
	$ sudo mkdir /opt/main
	$ sudo cp /ders/kutuphane/libdeneme.so /opt/main/
	$ sudo chmod 0755 /opt/main/libdeneme.so
	$ ./main 
 	Merhaba Dünya

Kütüphaneyi Uygulama İçine Gömme(Static Derleme)
++++++++++++++++++++++++++++++++++++++++++++++++

Bazı durumlarda ise kütüphane dosyalarını proje içine gömmek isteyebiliriz. **main** uygulamamız bağımlılığı olmayan bir uygulama yapabiliriz.
Bunun için;

.. code-block:: shell
	
	$ gcc -c -Wall -Werror -fpic deneme.c
	$ gcc -c main.c
	$ gcc main.o ./deneme.o -o main -static
	$ ldd ./main
	özdevimli bir çalıştırılabilir değil
	$ ./main 
	Merhaba Dünya
	
	
.. raw:: pdf

   PageBreak


