---
title: 《Java SE 8 for the really impatient》第二章习题笔记
date: 2017-02-15 21:38:25
tags:
- java
categories:
- 技术
---
3.

```
            long startTime = System.nanoTime();
			long countNotParallel = words.stream().count();
			long endTime = System.nanoTime();
			long time = endTime-startTime;
			System.out.println("stream方法：" + time + " 统计单词数为：" + countNotParallel);
			
			startTime = System.nanoTime();
			long countParallel = words.parallelStream().count();
			endTime = System.nanoTime();
			time = endTime - startTime;
			System.out.println("streamParallel方法：" + time + " 统计单词数为：" + countParallel);
			
			startTime = System.nanoTime();
			words.parallelStream();
			endTime = System.nanoTime();
			time = endTime - startTime;
			System.out.println("stream生成耗时：" + time);
```

顺便做了个stream生成耗时测试，运行结果可以看出stream是延时操作，只有当聚合方法被调用才会被执行。

---

6.

```
	public static void main(String[] args) {
		String string = "feng jing zhe da shuai b";
		characterStream(string).filter(c -> c != ' ').forEach(System.out::print);
	}
	
	public static Stream<Character> characterStream(String s) {
		return Stream.iterate(0, n->n+=1).limit(s.length()).map(s::charAt);
	}
```

注意这里是对象的方法引用。

---

9.

```
	public static void main(String[] args) {
		String string = "feng jing zhe";
		ArrayList<String> stringList = new ArrayList<String>(Arrays.asList(string.split(" ")));
		Stream<ArrayList<String>> stream = Stream.of(stringList, stringList, stringList, stringList);
		ArrayList<String> concatList = concat(stream);
		concatList.forEach(System.out::print);
	}
	
	public static <T> ArrayList<T> concat(Stream<ArrayList<T>> stream) {
		Optional<ArrayList<T>> optional = stream.reduce(Test::arrayListConcat);
		return optional.get();
	}
	
	public static <T> ArrayList<T> arrayListConcat(ArrayList<T> prev, ArrayList<T> next) {
		ArrayList<T> temp = (ArrayList<T>) prev.clone();
		temp.addAll(next);
		return temp;
	}
```

注意reduce（聚合）和collect（收集）的区别，就是让Stream<ArrayList<T>>转换成ArrayList<T>和ArrayList<ArrayList<T>>的区别。

---

12.

```
			String content = new String(Files.readAllBytes(Paths.get(
					"C:/Users/Administrator/Desktop/WarAndPeace.txt")), "UTF-8");
			List<String> words = Arrays.asList(content.split("[\\P{L}]+"));
			Stream<String> stream = words.stream();
			AtomicInteger[] wordLengthNum = new AtomicInteger[5];
			for (int i = 0; i < wordLengthNum.length; i++) {
				AtomicInteger atomicInteger = new AtomicInteger(0);
				wordLengthNum[i] = atomicInteger;
			}
			stream.parallel().forEach(s->{
				if (s.length() <= 5) {
					wordLengthNum[s.length()-1].incrementAndGet();
				}
			});
			for (AtomicInteger atomicInteger : wordLengthNum) {
				System.out.println(atomicInteger);
			}
```

---

13.

```
Map<Integer, Long> result = stream.collect(Collectors.groupingBy(String::length, Collectors.counting()));
```