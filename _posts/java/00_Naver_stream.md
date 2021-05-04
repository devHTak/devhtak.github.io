#### Problem01

```java
public class Problem01 {
	public static void main(String[] args) {
		int[] A = {100, 100, -10, -20, -30};
		String[] D = {"2020-01-01", "2020-02-01", "2020-02-11", "2020-02-05", "2020-02-08"};
		
		int[] numberOfPayments = new int[13];
		int[] totalPayOfPayments = new int[13];
		
//		int sum = 0;
//		for(int i = 0; i < D.length; i++) {
//			int month = Integer.parseInt(D[i].split("-")[1]);
//			
//			if(A[i] < 0) {
//				numberOfPayments[month] += 1;
//				totalPayOfPayments[month] += A[i];
//			}
//			sum += A[i];
//		}
		
		long sum = Stream.iterate(0, (a) -> a + 1).limit(D.length)
			.map(index -> new Integer[] {Integer.parseInt(D[index].split("-")[1]), index})
			.map(intArray -> {
				int index = intArray[1];
				int month = intArray[0];
				if(A[index] < 0) {
					numberOfPayments[month] += 1;
					totalPayOfPayments[month] += A[index];
				}
				return A[index];
			}).reduce(0, Integer::sum);
		System.out.println(sum);
		
		int deductedFee = Stream.iterate(1, i -> i + 1).limit(12)
			.filter(i -> numberOfPayments[i] < 3 || totalPayOfPayments[i] > -100)
			.reduce(0, (a, b) -> a + 1);
		
		System.out.println(deductedFee);
					
//		int deductedFee = 0;
//		for(int i = 1; i < 13; i++) {
//			if(numberOfPayments[i] >= 3 && totalPayOfPayments[i] <= -100) {
//				continue;
//			}
//			deductedFee += 1;
//		}
		
		System.out.println(sum);
		sum -= (5 * deductedFee);
		System.out.println(sum);		
	}
}
```

#### Problem02

```java
public class Problem02 {
	public static void main(String[] args) {
		int[] T = {2, -3, 3, 1, 10, 8, 2, 5, 13, -5, 3, -18};
		int n = T.length;
		int divideNum = n / 4;
		
		int[] maxMin = {-1001, 1001, -1001, 1001, -1001, 1001,-1001, 1001};
		for(int i = 0; i < n; i++) {
			if(i < divideNum) {
				maxMin[0] = getMax(maxMin[0], T[i]);
				maxMin[1] = getMin(maxMin[1], T[i]);
			} else if(i < divideNum * 2) {
				maxMin[2] = getMax(maxMin[2], T[i]);
				maxMin[3] = getMin(maxMin[3], T[i]);				
			} else if(i < divideNum * 3) {
				maxMin[4] = getMax(maxMin[4], T[i]);
				maxMin[5] = getMin(maxMin[5], T[i]);				
			} else if(i < divideNum * 4) {
				maxMin[6] = getMax(maxMin[6], T[i]);
				maxMin[7] = getMin(maxMin[7], T[i]);				
			}
		}
		
		String answer = getAnswer(maxMin);
		System.out.println(answer);
	}
	
	public static int getMax(int a, int b) {
		return a > b ? a : b;
	}
	
	public static int getMin(int a, int b) {
		return a > b ? b: a;
	}
	
	public static String getAnswer(int[] maxMin) {
		int max = -2001;
		int index = 0;
//		for(int i = 0; i <4; i++) {
//			int tmp = maxMin[i*2] - maxMin[i*2+1];
//			System.out.println(i + " " + tmp);
//			if(max < tmp) {
//				max = tmp;
//				index = i;
//			}
//		}
		
		Integer[] ans = Stream.iterate(0, i -> i + 1).limit(4)
			.map(i -> new Integer[] {maxMin[i * 2] - maxMin[i*2+1], i})
			.filter(tmp -> max < tmp[0])
			.reduce( (total, n) -> n).get();

		System.out.println(ans[0] + " " + ans[1]);
		
		if(ans[1] == 0) {
			return "WINTER";
		} else if(ans[1] == 1) {
			return "SPRING";
		} else if(ans[1] == 2) {
			return "SUMMER";
		} else {
			return "AUTUMN";
		}
	}
}
```
