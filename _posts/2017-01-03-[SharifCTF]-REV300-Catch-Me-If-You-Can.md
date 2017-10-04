---
title: "[SharifCTF] REV300 Catch Me If You Can"
tags: [reverse]
---

This challenge was pretty intresting since I didn't foud the flag in the first 30 seconds like the 3 previous one.<br/>
Okay, first things ou can download the sample here : [Login.apk.tar.xz](http://google.com)<br/>

Right, an APK ! an Android Package  !<br/>


Thirst thing I did i to disassemble it, with dex2jar and jd-gui.<br/>

So get a look at the first class ( you can see the  ): 

{% highlight java %}

package sharif.cert.ctf;

import android.view.View;
import android.view.View.OnClickListener;
import android.widget.EditText;
import android.widget.TextView;

class a
  implements View.OnClickListener
{
  a(MainActivity paramMainActivity) {}
  
  public void onClick(View paramView)
  {
    new String(" ");
    paramView = this.a.b.getText().toString(); // Retrieve user input
    paramView = this.a.a(paramView); 
    int i = this.a.processObject(paramView); // Call a strange method (processObject())
    if ((i == 1) && (this.a.e != "unknown")) {
      this.a.c.setText("Congratulations!");
    }
    if ((i == 1) && (this.a.e == "unknown")) {
      this.a.c.setText("Just keep Trying :-)");
    }
    if (i == 0) {
      this.a.c.setText("Just keep Trying :-)");
    }
  }
}


{% endhighlight %}

We can see that the ``` processObject() ``` method is not declared inside all the class we found .. But look at this class :

{% highlight java %}

package sharif.cert.ctf;

import android.app.Activity;
import android.os.Build;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

public class MainActivity
  extends Activity
{
  Button a;
  EditText b;
  TextView c;
  int d = 123;
  String e = "Code";
  
  static
  {
    System.loadLibrary("validation"); // What is this ?
  }
  
  public String a(String paramString)
  {
    return new HidingUtil().hide(paramString); // get a look at HidingUtils()
  }
  
  protected void onCreate(Bundle paramBundle)
  {
    super.onCreate(paramBundle);
    setContentView(2130968600);
    this.a = ((Button)findViewById(2131492942));
    this.b = ((EditText)findViewById(2131492941));
    this.c = ((TextView)findViewById(2131492943));
    this.e = Build.SERIAL;
    this.a.setOnClickListener(new a(this));
  }
  
  public native int processObject(String paramString);
}


{% endhighlight %}

Okay what is loadLibrary ? by looking at the fabulous Internet, I found some documentation about Loading native code to an Android Mobile Application.

So look at the HidingUtils code : 

{% highlight c  %}


#include <string.h>
#include <jni.h>
#include "Base64Util.h"
#include <android/log.h>

static unsigned char passwordKey[] = "My_S3cr3t_P@$$W0rD";

void xor_value_with_key(const char* value, char* xorOutput){
    int i = 0;
    while(value[i] != '\0'){
        int offset = i % sizeof(passwordKey);
        xorOutput[i] = value[i] ^ passwordKey[offset];
        i++;
    }
}

/**
 * com.apothesource.hidingpasswords.HidingUtil.hide
 *
 * This function uses a hard-coded password to XOR hide (encrypt) a provided message.
 */
jstring Java_com_apothesource_hidingpasswords_HidingUtil_hide(JNIEnv* env, jobject thiz, jstring 
javaString) {
    const char *nativeString = (*env)->GetStringUTFChars(env, javaString, 0);

    char xorOutput[BUFFFERLEN + 1] = "";
    xor_value_with_key(nativeString, xorOutput);

    char encodedoutput[BUFFFERLEN + 1] = "";

    Base64Encode(xorOutput, encodedoutput, BUFFFERLEN);

    (*env)->ReleaseStringUTFChars(env, javaString, nativeString);

    return (*env)->NewStringUTF(env, encodedoutput);

}

/**
 * com.apothesource.hidingpasswords.HidingUtil.unhide
 *
 * This function uses a hard-coded password to XOR unhide (decrypt) a provided message.
 */
jstring Java_com_apothesource_hidingpasswords_HidingUtil_unhide(JNIEnv* env, jobject thiz, 
jstring javaString) {
    const char *nativeString = (*env)->GetStringUTFChars(env, javaString, 0);

    char decodedoutput[BUFFFERLEN + 1] = "";

    Base64Decode(nativeString, decodedoutput, BUFFFERLEN);

    char xorOutput[BUFFFERLEN + 1] = "";
    xor_value_with_key(decodedoutput, xorOutput);

    (*env)->ReleaseStringUTFChars(env, javaString, nativeString);

    return (*env)->NewStringUTF(env, xorOutput);
}


{% endhighlight %}

A Simple xor function sets tho ! So now we know that the java application contains some .so for HidingUtils.<br/>

I did a simple binwalk extracting and I found something intresting (by extracting Login.apk with binwalk):

```
[sakiir@SakiiR-PC test]$ file _Login.apk.extracted/lib/x86_64/libhidingutil.so 
_Login.apk.extracted/lib/x86_64/libhidingutil.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=95bfd010a331f19445de63db36939c216aa32263, stripped
```

When I looked to the binary strings I found something relative to the encryption routine :


```
$ strings _Login.apk.extracted/lib/x86_64/libhidingutil.so


[...]
AUATI
[]A\A]A^A_
My_S3cr3t_P@$$W0rD
fx1uagMGQQMWOWhyFBxnBUdzN35NPWYHUBQHRmozeEY=
;*3$"
GCC: (GNU) 4.9.x 20150123 (prerelease)
gold 1.11
.shstrtab
.note.gnu.build-id
.dynsym
.dynstr
[...]

```

You can see that right after the Xor Encryption Key ( My_S3cr3t_P@$$W0rD ), a base64 string is present ! we have to find out why ! <br/>

```
$ echo "fx1uagMGQQMWOWhyFBxnBUdzN35NPWYHUBQHRmozeEY=" | base64 -d | xxd

00000000: 7f1d 6e6a 0306 4103 1639 6872 141c 6705  ..nj..A..9hr..g.
00000010: 4773 377e 4d3d 6607 5014 0746 6a33 7846  Gs7~M=f.P..Fj3xF
```

mmmh nothing readable, let's decrypt it with the super secret key ( My_S3cr3t_P@$$W0rD ) :

{% highlight python %}

#!/usr/bin/python2

key = "My_S3cr3t_P@$$W0rD\x00"
flag = "fx1uagMGQQMWOWhyFBxnBUdzN35NPWYHUBQHRmozeEY=".decode("base64")

out = ""
i = 0
for c in flag:
    out += chr(ord(c) ^ ord(key[i % len(key)]))
    i += 1

print "SharifCTF{" + out + "}"

{% endhighlight %}

This code give us the flag ;)


