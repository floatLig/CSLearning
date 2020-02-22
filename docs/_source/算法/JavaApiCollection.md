# Collection

## 测试用例

```java
public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Collections.addAll(list, "a", "c", "b");
        System.out.println("添加元素" + list);
        //print：添加元素[a, c, b]

        //打乱顺序
        Collections.shuffle(list);
        System.out.println("打乱顺序：" + list);
        //print：打乱顺序：[a, b, c]

        //重新排序
        Collections.sort(list);
        System.out.println("排序：" + list);
        //print：排序：[a, b, c]

        Person p2 = new Person("李四", 19);
        Person p1 = new Person("张三", 19);
        Person p3 = new Person("张三", 20);
        Person p4 = new Person("王五", 21);

        List<Person> personList = new ArrayList<>();
        Collections.addAll(personList, p1, p2, p3, p4);
        Collections.sort(personList);

        System.out.println("自定义类排序: " + personList);
        //print：自定义类排序: [Person{name='张三', age=19}, Person{name='李四', age=19}, Person{name='张三', age=20}, Person{name='王五', age=21}]
    }

    static class Person implements Comparable<Person> {
        String name;
        Integer age;

        Person(String name, Integer age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public int compareTo(Person o) {
            /**
             * return 0：认为元素都是相同的
             * 自定义比较的规则，比较两个人的年龄（this，参数Person）
             * this.getAge() - o.getAge(): 年龄升序排序
             */
            int result = this.age - o.age;
            if (result == 0) {
                result = this.name.charAt(0) - o.name.charAt(0);
            }
            return result;
        }

        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
```