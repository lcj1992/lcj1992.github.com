---
layout: post
title: java native调用
date: 2016-02-28
categories: java
---



### Object

*   registerNatives()
*   getClass()
*   hashCode()
*   equals()
*   clone()
*   toString()
*   notify()
*   notifyAll()
*   wait(long timeout)
*   wait(long timeout, int nanos)
*   wait()
*   finalize()

Object就这几个方法,其中除equals和toString,其他都是native方法. 

本文就以Object来探讨下java的native,源码基于openjdk7

Object native方法的声明在openjdk/jdk/src/share/native/java/lang/Object.c中

    static JNINativeMethod methods[] = {
        {"hashCode",    "()I",                    (void *)&JVM_IHashCode},
        {"wait",        "(J)V",                   (void *)&JVM_MonitorWait},
        {"notify",      "()V",                    (void *)&JVM_MonitorNotify},
        {"notifyAll",   "()V",                    (void *)&JVM_MonitorNotifyAll},
        {"clone",       "()Ljava/lang/Object;",   (void *)&JVM_Clone},
    };

JNINativeMethod的结构体如下: 
 
    typedef struct {
        char *name;
        char *signature;
        void *fnPtr;
    } JNINativeMethod;
    
*   name  Object的方法名称
*   signature 方法签名
*   fnPtr native实现的函数指针

对应的native实现在openjdk/hotspot/src/share/vm/prims/jvm.cpp

hashCode():

    JVM_ENTRY(jint, JVM_IHashCode(JNIEnv* env, jobject handle))
      JVMWrapper("JVM_IHashCode");
      // as implemented in the classic virtual machine; return 0 if object is NULL
      return handle == NULL ? 0 : ObjectSynchronizer::FastHashCode (THREAD, JNIHandles::resolve_non_null(handle)) ;
    JVM_END

wait():

    JVM_ENTRY(void, JVM_MonitorWait(JNIEnv* env, jobject handle, jlong ms))
      JVMWrapper("JVM_MonitorWait");
      Handle obj(THREAD, JNIHandles::resolve_non_null(handle));
      assert(obj->is_instance() || obj->is_array(), "JVM_MonitorWait must apply to an object");
      JavaThreadInObjectWaitState jtiows(thread, ms != 0);
      if (JvmtiExport::should_post_monitor_wait()) {
        JvmtiExport::post_monitor_wait((JavaThread *)THREAD, (oop)obj(), ms);
      }
      ObjectSynchronizer::wait(obj, ms, THREAD);
    JVM_END
    
...

JVM_ENTRY的结构体定义在openjdk/hotspot/src/share/vm/runtime/interfaceSupport.hpp
    
    #define JVM_ENTRY(result_type, header)                               \
    extern "C" {                                                         \
      result_type JNICALL header {                                       \
        JavaThread* thread=JavaThread::thread_from_jni_environment(env); \
        ThreadInVMfromNative __tiv(thread);                              \
        debug_only(VMNativeEntryWrapper __vew;)                          \
        VM_ENTRY_BASE(result_type, header, thread)
        
JVM_END

    #define JVM_END } }
