# 운영 환경에서 System.out.println() 을 사용하면 안되는 이유

## System.out.println()

```java
public class PrintStream extends FilterOutputStream
	implements Appendable, Closeable {
	/**
	 * Terminates the current line by writing the line separator string.  The
	 * line separator string is defined by the system property
	 * {@code line.separator}, and is not necessarily a single newline
	 * character ({@code '\n'}).
	 */
	public void println() {
		newLine();
	}

	private void newLine() {
		try {
			synchronized (this) {
				ensureOpen();
				textOut.newLine();
				textOut.flushBuffer();
				charOut.flushBuffer();
				if (autoFlush)
					out.flush();
			}
		} catch (InterruptedIOException x) {
			Thread.currentThread().interrupt();
		} catch (IOException x) {
			trouble = true;
		}
	}
}
```

위의 코드를 확인해보면, `newLine()` 메서드 안에서 `synchronized` 키워드를 사용하고 있다. 멀티 스레드 환경에서 `특정 스레드` 가 `newLine()` 을 호출하게 되면,
`newLine()` 을 사용하고자 하는 `다른 스레드` 들은 모두 `Block` 상태에 빠지게 된다. 이로 인해 극심한 성능 저하가 발생할 수 있기 때문에 상용 환경에서는 `System.out.println()`
를 절대로 사용하면 안된다.

> 한 번 요청 시 5000명의 사용자를 요청하고, 처리 과정에서 응답시간이 20초 걸리는 사이트가 있는데,
> 원인을 알아보니 5000명의 정보를 다 System.out.println()으로 처리하고있던 것이다.
> 이는 System.out.println()을 줄임으로써 응답시간이 6초까지 줄었다.
> - 이상민, 자바 성능 튜닝이야기, 인사이트, 2013

## 예상 면접 질문

- `synchronized` 랑 `Blocking IO` 가 만나면 어떻게 환장의 성능 하락을 만들 수 있는걸까요?
- 이 두 개가 만났을 때 스레드가 어떻게 동작할지, CPU 사용률은 어떻게 될지 시뮬레이션을 해보세요.
    - 답변) 하나의 스레드마다 작업을 처리하기 때문에 그동안 CPU 많은 작업을 하지 않으므로 사용률이 떨어질 것으로 예상된다.
