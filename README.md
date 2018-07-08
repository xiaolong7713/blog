# 那些年踩过的Java的坑
持续更新ing

------

在Java开发的工作或学习中难免会踩坑，为此整理出一些平常中会遇到的案例，方便后来的小伙伴去避免，欢迎大家可以一起参与分享交流。

------
**Last Update 2018.7.8**

-----
### 1. Jdk中String.valueOf(Object obj)并不能返回你想要的结果
下面的这段代码
``` java
public static void main(String[] args) {
	Integer i = null;
	String str = String.valueOf(i);
	System.out.println(str);
}
```
你会认为会输出空串或者空对象，实际上返回了一个'null'字符串，观看源码，会发现
``` java
public static String valueOf(Object obj) {
  return (obj == null) ? "null" : obj.toString();
}
```
这段源码，从jdk1.6到1.8没有修改，也许是作者不愿意返回空对象。

### 2. Jdk中Collections类中提供的empty集合不能直接添加值
下面的这段代码：
``` java
static class StudentDto {
		private int age;
		private List<Integer> sorce = Collections.emptyList();

		public int getAge() {
			return age;
		}

		public void setAge(int age) {
			this.age = age;
		}

		public List<Integer> getSorce() {
			return sorce;
		}

		public void setSorce(List<Integer> sorce) {
			this.sorce = sorce;
		}
	}

	public static void main(String[] args) {
		// 或者其他方式获得的Dto，这里的new只是举例
		StudentDto dto = new StudentDto();
		if (dto.getSorce() != null) {
			dto.getSorce().add(96);
		}
	}
```
这段代码不能正常在List当中添加值，实际它返回一个异常
``` java
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:131)
	at java.util.AbstractList.add(AbstractList.java:91)
	at com.iflytek.Test.main(Test.java:33)
```
Collections工具类提供了一些集合的静态操作方法，还提供了一些empty集合，这些空集合是一个真正的实例，但size都为0，不能进行删除和插入操作，以emptyList()为例，
```java
private static class EmptyList
	extends AbstractList<Object>
	implements RandomAccess, Serializable {
	// use serialVersionUID from JDK 1.2.2 for interoperability
	private static final long serialVersionUID = 8842843931221139166L;

        public int size() {return 0;}

        public boolean contains(Object obj) {return false;}

        public Object get(int index) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }

        // Preserves singleton property
        private Object readResolve() {
            return EMPTY_LIST;
        }
    }
```
继承于AbstractList抽象类，并没有重写add和remove等方法。

### 3. Jdk中的Array.asList()方法不能直接添加值
看下面这段代码：
``` java
public class Demo {

	public static void main(String[] args) {
		//将String数组转成List
		String[] str = new String[] { "张三", "李四" };
		List<String> list = Arrays.asList(str);
		list.add("王二");
	}
}
```
这段代码不能正常在List当中添加值，实际它返回一个异常
``` java
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
	at com.iflytek.sgy.zhyl.web.Demo.main(Demo.java:11)
```
实际上，Arrays.asList()方法返回的ArrayList对象不是我们常用的java.util.ArrayList类，这里的ArrayList类是Arrays类中一个内部类，它继承AbstractList抽象类，没有重写add方法，直接调用抽象类的中add方法会抛出异常。
``` java
 private static class ArrayList<E> extends AbstractList<E>
        implements RandomAccess, java.io.Serializable
    {
        private static final long serialVersionUID = -2764017481108945198L;
        private final E[] a;

        ArrayList(E[] array) {
            if (array==null)
                throw new NullPointerException();
            a = array;
        }

        public int size() {
            return a.length;
        }

        public Object[] toArray() {
            return a.clone();
        }

        public <T> T[] toArray(T[] a) {
            int size = size();
            if (a.length < size)
                return Arrays.copyOf(this.a, size,
                                     (Class<? extends T[]>) a.getClass());
            System.arraycopy(this.a, 0, a, 0, size);
            if (a.length > size)
                a[size] = null;
            return a;
        }

        public E get(int index) {
            return a[index];
        }

        public E set(int index, E element) {
            E oldValue = a[index];
            a[index] = element;
            return oldValue;
        }

        public int indexOf(Object o) {
            if (o==null) {
                for (int i=0; i<a.length; i++)
                    if (a[i]==null)
                        return i;
            } else {
                for (int i=0; i<a.length; i++)
                    if (o.equals(a[i]))
                        return i;
            }
            return -1;
        }

        public boolean contains(Object o) {
            return indexOf(o) != -1;
        }
    }
```
