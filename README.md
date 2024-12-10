
# 前言


本文可在[https://paw5zx.github.io/GObject\-tutorial\-beginner\-02/](https://github.com)中阅读，体验更加


在[上一节](https://github.com)中我们介绍了GObject类型的类和实例变量的创建和使用。GObject是一个基本的可实例化类类型，是所有使用GObject系统的类型的基类，提供了继承、封装、多态等面向对象的核心特性。不过我们一般不直接使用GObject本身，而是通过继承GObject来创建新的类型。


{% notel blue fa\-circle\-exclamation 注意：%}
对于GObject官方文档中：


可实例化的类类型：object（Instantiatable classed types: objects），本文以及后文中称其为类型。
不可实例化的类类型：interface（Non\-instantiatable classed types: interfaces），本文以及后文中称其为接口。


本文中我们主要介绍一下GObject类型系统中的命名约定，以及介绍如何创建一个自定义的类型。


# 命名约定


{% note blue fa\-circle\-exclamation %}


命名约定贯穿整个GObject类型系统。


{% endnote %}


首先，我们先了解基本的命名约定：一个对象名由命名空间（也可称为moudle）和名称组成。例如，GObject由命名空间“G”和名称“Object”组成。GtkWidget由命名空间“Gtk”和名称“Widget”组成。在本文中，为了更好的演示，我们将定义一个新类型，其命名空间为“Paw”，名称为“Double”，用于表示一个浮点数据。


好了，你现在已了解了基本的命名约定，下面我们来看一下更多的命名约定：


* 类型名称必须至少有三个字符长，并以a\-z， A\-Z或`_`开头。
* 函数名使用object\_method模式：要在类型为`Double`的实例上定义名为`add`的函数，则函数名为`double_save`。
* 使用前缀可以避免与其他项目的名称空间冲突。如我的库（或应用程序）名为`Paw`，就在所有函数名前加上`paw_`。例如：`paw_double_add`。前缀应该是一个词，即在第一个字母之后不应该包含任何大写字母。例如，应该是Exampleprefix而不是ExamplePrefix。


除了上述约定，本文还将介绍其他命名约定，将在文中适时描述。


# 类型创建和注册


类型的创建是指定义一个新的GObject类型，包括其所有的数据和行为。这会涉及到：


* 定义类结构体和实例结构体
* 定义类型相关的函数：如类初始化（`class_init`）、实例初始化（`instance_init`）等函数。


类型的注册是将创建的新GObject类型在GObject类型系统中注册，使其成为GObject系统可识别和使用的一部分。这会涉及到：


* 调用类型注册函数：`g_type_register_static`或`g_type_register_dynamic`，这些函数负责在GObject类型系统中注册新创建的类型


在我们平时的使用中，类型的创建和注册通常可以通过GObject提供的一些便利宏来完成，如`G_DECLARE_FINAL_TYPE`和`G_DEFINE_TYPE`等。这些便利宏可以让用户不用关心一些类型创建和注册的具体细节。


在本文中，为了更好地理解GObject类型系统，会先展示不使用便利宏的类型创建和注册过程，之后再介绍便利宏。


{% notel blue fa\-circle\-exclamation 注意：%}
GObject类型系统中有两种类型：静态类型和动态类型。


对于静态类型，即使所有实例都被销毁后也不会销毁其类。对于动态类型，在最后一个实例被销毁时销毁其类。GObject的类型是静态的，其派生的类型也是静态的。


## 手动


### 定义类型的类和实例结构体


类和实例结构体命名约定如下：


* 类结构体的命名为：`Class`，如PawDoubleClass。
* 实例结构体的命名为：，如PawDouble。



```


|  | //类结构体 |
| --- | --- |
|  | typedef struct _PawDoubleClass PawDoubleClass; |
|  | struct _PawDoubleClass |
|  | { |
|  | GObjectClass parent_class; |
|  | }; |


```

PawDoubleClass的第一个成员必须是其父类型的类结构体



```


|  | //实例结构体 |
| --- | --- |
|  | typedef struct _PawDouble PawDouble; |
|  | struct _PawDouble |
|  | { |
|  | GObject parent; |
|  | double value; |
|  | }; |


```

PawDouble的第一个成员必须是其父类型的实例结构体。
PawDouble有自己的成员`value`，是PawDouble类型代表的浮点数据的值


### 定义初始化函数


类和实例的初始化函数命名约定如下：


* 类初始化函数的命名为：`__class_init`，如paw\_double\_class\_init。
* 实例初始化函数的命名为：`__init`，如paw\_double\_init。



```


|  | //类构造函数 |
| --- | --- |
|  | static void paw_double_class_init(PawDoubleClass* class) |
|  | { |
|  | } |
|  |  |
|  | //实例构造函数 |
|  | static void paw_double_init(PawDouble* self) |
|  | { |
|  | } |


```

### 类型注册


对于静态类型，我们使用`g_type_register_static`将其注册至GObject的类型系统中



```


|  | //file: gtype.h |
| --- | --- |
|  | GType |
|  | g_type_register_static (GType            parent_type, |
|  | const gchar*     type_name, |
|  | const GTypeInfo* info, |
|  | GTypeFlags       flags); |


```

其中：


* parent\_type：父类类型
* type\_name：类型名称，如PawDouble
* info：向类型系统传递类型初始化和销毁的相关信息。GTypeInfo结构体将在下文中介绍
* flags：如果类型是抽象类型或抽象值类型，则设置它们的标志位。否则，将其设为0。


GTyepInfo结构体：



```


|  | typedef struct _GTypeInfo  GTypeInfo; |
| --- | --- |
|  |  |
|  | struct _GTypeInfo |
|  | { |
|  | /* interface types, classed types, instantiated types */ |
|  | guint16                class_size; |
|  |  |
|  | GBaseInitFunc          base_init; |
|  | GBaseFinalizeFunc      base_finalize; |
|  |  |
|  | /* interface types, classed types, instantiated types */ |
|  | GClassInitFunc         class_init; |
|  | GClassFinalizeFunc     class_finalize; |
|  | gconstpointer          class_data; |
|  |  |
|  | /* instantiated types */ |
|  | guint16                instance_size; |
|  | guint16                n_preallocs; |
|  | GInstanceInitFunc      instance_init; |
|  |  |
|  | /* value handling */ |
|  | const GTypeValueTable  *value_table; |
|  | }; |


```

此结构必须在类型注册前创建，其中：


* `class_size`: 类的大小。例如，PawDouble类型的类大小为`sizeof(PawDoubleClass)`。
* `base_init`, `base_finalize`: 这些函数初始化/销毁类的动态成员。在许多情况下，它们不是必需的，被赋值为NULL。详见[GObject.BaseInitFunc](https://github.com):[milou云加速器官网](https://jiechuangmoxing.com)和[GObject.ClassInitFunc](https://github.com)。
* `class_init`: 类的初始化函数。用户需要将定义的类初始化函数赋值给`class_init`成员。按照命名约定，类初始化函数名称为`__class_init`，例如`paw_double_class_init`。
* `class_finalize`: 类的销毁函数。因为GObject的子类类型是静态的，所以它没有销毁函数。将`class_finalize`成员赋值为NULL即可。
* `class_data`: 用户提供数据，传递给类的初始化/销毁函数。通常赋值为 NULL。
* `instance_size`: 实例的大小。例如，PawDouble类型的实例大小为`sizeof(PawDouble)`。
* `n_preallocs`: 用于指定预分配的实例数量。自GLib 2\.10以来，该字段被忽略。
* `instance_init`: 实例的初始化函数。用户需要将定义的实例初始化函数赋值给`instance_init`成员。按照命名约定，实例初始化函数名称为`__init`，例如 `paw_double_init`。
* `value_table`: 这通常只对基本类型有用。如果类型是GObject的后代，赋值为NULL。


我们将编写一个函数`paw_double_get_type`，用于向GObject类型系统注册新类型，并返回所注册类型的id



```


|  | GType paw_double_get_type(void) |
| --- | --- |
|  | { |
|  | static GType type = 0; |
|  | GTypeInfo info; |
|  |  |
|  | if(type == 0) |
|  | { |
|  | //赋值 |
|  | info.class_size = sizeof(PawDoubleClass); |
|  | info.base_init = NULL; |
|  | info.base_finalize = NULL; |
|  | info.class_init = (GClassInitFunc) paw_double_class_init; |
|  | info.class_finalize = NULL; |
|  | info.class_data = NULL; |
|  | info.instance_size = sizeof(PawDouble); |
|  | info.n_preallocs = 0; |
|  | info.instance_init = (GInstanceInitFunc) paw_double_init; |
|  | info.value_table = NULL; |
|  | //注册类型 |
|  | type = g_type_register_static(G_TYPE_OBJECT, "PawDouble", &info, 0); |
|  | } |
|  | return type; |
|  | } |


```

### 汇总


上述的类型创建和注册过程汇总如下：



```


|  | #include |
| --- | --- |
|  |  |
|  | #define PAW_TYPE_DOUBLE  (paw_double_get_type()) |
|  |  |
|  | //定义实例结构体 |
|  | typedef struct _PawDouble PawDouble; |
|  | struct _PawDouble |
|  | { |
|  | GObject parent; |
|  | double value; |
|  | }; |
|  |  |
|  | //定义类结构体 |
|  | typedef struct _PawDoubleClass PawDoubleClass; |
|  | struct _PawDoubleClass |
|  | { |
|  | GObjectClass parent_class; |
|  | }; |
|  |  |
|  | //类构造函数 |
|  | static void paw_double_class_init(PawDoubleClass* class) |
|  | { |
|  | } |
|  |  |
|  | //实例构造函数 |
|  | static void paw_double_init(PawDouble* self) |
|  | { |
|  | } |
|  |  |
|  | //首次调用时注册类型 |
|  | //返回类型 |
|  | GType paw_double_get_type(void) |
|  | { |
|  | static GType type = 0; |
|  | GTypeInfo info; |
|  |  |
|  | if(type == 0) |
|  | { |
|  | info.class_size = sizeof(PawDoubleClass); |
|  | info.base_init = NULL; |
|  | info.base_finalize = NULL; |
|  | info.class_init = (GClassInitFunc) paw_double_class_init; |
|  | info.class_finalize = NULL; |
|  | info.class_data = NULL; |
|  | info.instance_size = sizeof(PawDouble); |
|  | info.n_preallocs = 0; |
|  | info.instance_init = (GInstanceInitFunc) paw_double_init; |
|  | info.value_table = NULL; |
|  | type = g_type_register_static(G_TYPE_OBJECT, "PawDouble", &info, 0); |
|  | } |
|  | return type; |
|  | } |
|  |  |
|  | int main(int argc, char **argv) |
|  | { |
|  | GType dtype; |
|  | PawDouble* d; |
|  |  |
|  | dtype = paw_double_get_type(); /* or dtype = PAW_TYPE_DOUBLE */ |
|  | if(dtype) |
|  | g_print("Registration was a success. The type is %lx.\n", dtype); |
|  | else |
|  | g_print("Registration failed.\n"); |
|  |  |
|  | d = g_object_new(PAW_TYPE_DOUBLE, NULL); |
|  | if(d) |
|  | g_print("Instantiation was a success. The instance address is %p.\n", d); |
|  | else |
|  | g_print("Instantiation failed.\n"); |
|  | g_object_unref(d); /* Releases the object d. */ |
|  |  |
|  | return 0; |
|  | } |


```

* 20\-28：类初始化函数和实例初始化函数，参数`class`是指类结构体，参数`self`是指实例结构体。这两个函数在示例中内容为空，但他们是注册过程必须的。
* 30\-52：paw\_double\_get\_type函数，返回PawDouble类型的id。这种函数的命名总是遵循`__get_type`。同时，宏`_TYPE_`（所有字符都是大写）是这个函数的别名。paw\_double\_get\_type函数有一个静态变量`type`来存储对象的类型。在此函数的第一次调用时，`type`是零。然后函数内调用 `g_type_register_static`将新类型注册到类型系统中。在第二次和之后的调用中，该函数只是返回`type`，因为静态变量`type`已经被`g_type_register_static`赋予了非零值，并且它保持该值。
* 39\-49：设置`info`结构体并调用`g_type_register_static`注册。
* 54\-73：主函数。获取PawDouble类型id并显示它。函数`g_object_new`被用来创建类型实例。GObject API文档中表示`g_object_new`返回一个指向GObject实例的指针，但实际上它返回的是一个`gpointer`。`gpointer`基本等同于`void*`，可以被赋值给指向任何类型的指针。因此，语句`d = g_object_new(PAW_TYPE_DOUBLE, NULL);`是正确的。如果函数`g_object_new`返回的是`GObject*`，那么就需要对返回的指针进行类型转换。在创建完成后，我们打印了实例的地址。最后，使用函数`g_object_unref`释放并销毁实例。


剩余内容请前往[https://paw5zx.github.io/GObject\-tutorial\-beginner\-02/](https://github.com)中阅读。


