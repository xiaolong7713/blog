# 那些年踩过的Java的坑
持续更新ing

------

在Java开发的工作或学习中难免会踩坑，为此整理出一些平常中会遇到的案例，方便后来的小伙伴去避免，欢迎大家可以一起参与分享交流。

------
**Last Update 2018.6.30**


------
1.[Jdk中String.valueOf(Object obj)并不能返回你想要的结果](https://github.com/xiaolong7713/blog/edit/master/README.md#1.Jdk中String.valueOf(Object obj)并不能返回你想要的结果)
2. [Jdk中Collections类中提供的empty集合不能直接添加值](https://github.com/xiaolong7713/blog/edit/master/README.md#2.Jdk中Collections类中提供的empty集合不能直接添加值)
3. [百度](https://www.baidu.com)

-----

### 1.Jdk中String.valueOf(Object obj)并不能返回你想要的结果
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

### 2.Jdk中Collections类中提供的empty集合不能直接添加值
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
