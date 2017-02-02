---
layout: post
title: Internal value representation in PHP 7 - Part 2
excerpt: Covers the implementation of complex types like strings and objects in PHP 7, as well as a number of special types like indirect zvals.
---
In the [first part][part1] of this article, high level changes in the internal value representation between PHP 5 and
PHP 7 were discussed. As a reminder, the main difference was that zvals are no longer individually allocated and don't
store a reference count themselves. Simple values like integers or floats can be stored directly in a zval, while
complex values are represented using a pointer to a separate structure.

The additional structures for complex zval values all use a common header, which is defined by `zend_refcounted`:

{% highlight c %}
struct _zend_refcounted {
    uint32_t refcount;
    union {
        struct {
            ZEND_ENDIAN_LOHI_3(
                zend_uchar    type,
                zend_uchar    flags,
                uint16_t      gc_info)
        } v;
        uint32_t type_info;
    } u;
};
{% endhighlight %}

This header now holds the `refcount`, the `type` of the value and cycle collection info (`gc_info`), as well as a slot
for type-specific `flags`.