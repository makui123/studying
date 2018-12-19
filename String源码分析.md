##String 源码分析
	
#String源码如下：
	
	package java.lang;
	
	import java.io.ObjectStreamField;
	import java.io.UnsupportedEncodingException;
	import java.nio.charset.Charset;
	import java.util.ArrayList;
	import java.util.Arrays;
	import java.util.Comparator;
	import java.util.Formatter;
	import java.util.Locale;
	import java.util.Objects;
	import java.util.StringJoiner;
	import java.util.regex.Matcher;
	import java.util.regex.Pattern;
	import java.util.regex.PatternSyntaxException;
	
	//final修饰的类String 不可以被修改，不能被继承  因为String对象是不可变的，它们可以被共享
	public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
		//String类实现了3个接口
		//1.java.io.Serializable  这个序列化接口没有任何方法和域，仅用于标识序列化的语意。(深入了解Java序列化)
		//2.Comparable<T>  只有一个public int compareTo(T o); 方法比较
		//3.CharSequence	这个接口是一个只读的字符序列。包括length(), charAt(int index), subSequence(int start, int end)这几个API接口，值得一提的是，StringBuffer和StringBuild也是实现了改接口。
	    /** The value is used for character storage. */
		//char数组用于存储内容 比如"abc"存储在char数组中
	    private final char value[];
	
	    /** Cache the hash code for the string */
		//而hash是String实例化的hashcode的一个缓存。因为String经常被用于比较，比如在HashMap中。如果每次进行比较都重新计算	hashcode的值的话，那无疑是比较麻烦的，而保存一个hashcode的缓存无疑能优化这样的操作。
	    private int hash; // Default to 0
	
	    //序列化标志
	    private static final long serialVersionUID = -6849794470754667710L;
			
	  	//
	    private static final ObjectStreamField[] serialPersistentFields =
	        new ObjectStreamField[0];
	
	 
	    public String() {
	        this.value = "".value;
	    }
	
	   
	    public String(String original) {
	        this.value = original.value;
	        this.hash = original.hash;
	    }
	
	   
	    public String(char value[]) {
	        this.value = Arrays.copyOf(value, value.length);
	    }
	
	  
	    public String(char value[], int offset, int count) {
	        if (offset < 0) {
	            throw new StringIndexOutOfBoundsException(offset);
	        }
	        if (count <= 0) {
	            if (count < 0) {
	                throw new StringIndexOutOfBoundsException(count);
	            }
	            if (offset <= value.length) {
	                this.value = "".value;
	                return;
	            }
	        }
	        // Note: offset or count might be near -1>>>1.
	        if (offset > value.length - count) {
	            throw new StringIndexOutOfBoundsException(offset + count);
	        }
	        this.value = Arrays.copyOfRange(value, offset, offset+count);
	    }
	
	   
	    public String(int[] codePoints, int offset, int count) {
	        if (offset < 0) {
	            throw new StringIndexOutOfBoundsException(offset);
	        }
	        if (count <= 0) {
	            if (count < 0) {
	                throw new StringIndexOutOfBoundsException(count);
	            }
	            if (offset <= codePoints.length) {
	                this.value = "".value;
	                return;
	            }
	        }
	        // Note: offset or count might be near -1>>>1.
	        if (offset > codePoints.length - count) {
	            throw new StringIndexOutOfBoundsException(offset + count);
	        }
	
	        final int end = offset + count;
	
	        // Pass 1: Compute precise size of char[]
	        int n = count;
	        for (int i = offset; i < end; i++) {
	            int c = codePoints[i];
	            if (Character.isBmpCodePoint(c))
	                continue;
	            else if (Character.isValidCodePoint(c))
	                n++;
	            else throw new IllegalArgumentException(Integer.toString(c));
	        }
	
	        // Pass 2: Allocate and fill in char[]
	        final char[] v = new char[n];
	
	        for (int i = offset, j = 0; i < end; i++, j++) {
	            int c = codePoints[i];
	            if (Character.isBmpCodePoint(c))
	                v[j] = (char)c;
	            else
	                Character.toSurrogates(c, v, j++);
	        }
	
	        this.value = v;
	    }
	
	    @Deprecated
	    public String(byte ascii[], int hibyte, int offset, int count) {
	        checkBounds(ascii, offset, count);
	        char value[] = new char[count];
	
	        if (hibyte == 0) {
	            for (int i = count; i-- > 0;) {
	                value[i] = (char)(ascii[i + offset] & 0xff);
	            }
	        } else {
	            hibyte <<= 8;
	            for (int i = count; i-- > 0;) {
	                value[i] = (char)(hibyte | (ascii[i + offset] & 0xff));
	            }
	        }
	        this.value = value;
	    }
	
	   
	    @Deprecated
	    public String(byte ascii[], int hibyte) {
	        this(ascii, hibyte, 0, ascii.length);
	    }
	
	  
	    private static void checkBounds(byte[] bytes, int offset, int length) {
	        if (length < 0)
	            throw new StringIndexOutOfBoundsException(length);
	        if (offset < 0)
	            throw new StringIndexOutOfBoundsException(offset);
	        if (offset > bytes.length - length)
	            throw new StringIndexOutOfBoundsException(offset + length);
	    }
	
	 
	    public String(byte bytes[], int offset, int length, String charsetName)
	            throws UnsupportedEncodingException {
	        if (charsetName == null)
	            throw new NullPointerException("charsetName");
	        checkBounds(bytes, offset, length);
	        this.value = StringCoding.decode(charsetName, bytes, offset, length);
	    }
	
	  
	    public String(byte bytes[], int offset, int length, Charset charset) {
	        if (charset == null)
	            throw new NullPointerException("charset");
	        checkBounds(bytes, offset, length);
	        this.value =  StringCoding.decode(charset, bytes, offset, length);
	    }
	
	   
	    public String(byte bytes[], String charsetName)
	            throws UnsupportedEncodingException {
	        this(bytes, 0, bytes.length, charsetName);
	    }
	
	    public String(byte bytes[], Charset charset) {
	        this(bytes, 0, bytes.length, charset);
	    }
	
	    public String(byte bytes[], int offset, int length) {
	        checkBounds(bytes, offset, length);
	        this.value = StringCoding.decode(bytes, offset, length);
	    }
	
	   
	    public String(byte bytes[]) {
	        this(bytes, 0, bytes.length);
	    }
	
	 	//StringBuffer作为构造参数 StringBuffer线程安全的，用到了synchronized
	    public String(StringBuffer buffer) {
	        synchronized(buffer) {
	            this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
	        }
	    }
	
	   	/StringBuilder作为构造参数
	    public String(StringBuilder builder) {
	        this.value = Arrays.copyOf(builder.getValue(), builder.length());
	    }
	
	    
	    String(char[] value, boolean share) {
	        // assert share : "unshared not supported";
	        this.value = value;
	    }
	
	  
	    public int length() {
	        return value.length;
	    }
		
	    //判断String对象的长度是否为0
	    public boolean isEmpty() {
	        return value.length == 0;
	    }
	
	   
	    public char charAt(int index) {
	        if ((index < 0) || (index >= value.length)) {
	            throw new StringIndexOutOfBoundsException(index);
	        }
	        return value[index];
	    }
	
	   
	    public int codePointAt(int index) {
	        if ((index < 0) || (index >= value.length)) {
	            throw new StringIndexOutOfBoundsException(index);
	        }
	        return Character.codePointAtImpl(value, index, value.length);
	    }
	
	  
	    public int codePointBefore(int index) {
	        int i = index - 1;
	        if ((i < 0) || (i >= value.length)) {
	            throw new StringIndexOutOfBoundsException(index);
	        }
	        return Character.codePointBeforeImpl(value, index, 0);
	    }
	
	  
	    public int codePointCount(int beginIndex, int endIndex) {
	        if (beginIndex < 0 || endIndex > value.length || beginIndex > endIndex) {
	            throw new IndexOutOfBoundsException();
	        }
	        return Character.codePointCountImpl(value, beginIndex, endIndex - beginIndex);
	    }
	
	  
	    public int offsetByCodePoints(int index, int codePointOffset) {
	        if (index < 0 || index > value.length) {
	            throw new IndexOutOfBoundsException();
	        }
	        return Character.offsetByCodePointsImpl(value, 0, value.length,
	                index, codePointOffset);
	    }
	
	    /**
	     * Copy characters from this string into dst starting at dstBegin.
	     * This method doesn't perform any range checking.
	     */
	    void getChars(char dst[], int dstBegin) {
	        System.arraycopy(value, 0, dst, dstBegin, value.length);
	    }
	
	   
	    public void getChars(int srcBegin, int srcEnd, char dst[], int dstBegin) {
	        if (srcBegin < 0) {
	            throw new StringIndexOutOfBoundsException(srcBegin);
	        }
	        if (srcEnd > value.length) {
	            throw new StringIndexOutOfBoundsException(srcEnd);
	        }
	        if (srcBegin > srcEnd) {
	            throw new StringIndexOutOfBoundsException(srcEnd - srcBegin);
	        }
	        System.arraycopy(value, srcBegin, dst, dstBegin, srcEnd - srcBegin);
	    }
	
	   
	    @Deprecated
	    public void getBytes(int srcBegin, int srcEnd, byte dst[], int dstBegin) {
	        if (srcBegin < 0) {
	            throw new StringIndexOutOfBoundsException(srcBegin);
	        }
	        if (srcEnd > value.length) {
	            throw new StringIndexOutOfBoundsException(srcEnd);
	        }
	        if (srcBegin > srcEnd) {
	            throw new StringIndexOutOfBoundsException(srcEnd - srcBegin);
	        }
	        Objects.requireNonNull(dst);
	
	        int j = dstBegin;
	        int n = srcEnd;
	        int i = srcBegin;
	        char[] val = value;   /* avoid getfield opcode */
	
	        while (i < n) {
	            dst[j++] = (byte)val[i++];
	        }
	    }
	
		
	    public byte[] getBytes(String charsetName)
	            throws UnsupportedEncodingException {
	        if (charsetName == null) throw new NullPointerException();
	        return StringCoding.encode(charsetName, value, 0, value.length);
	    }
	
	   
	    public byte[] getBytes(Charset charset) {
	        if (charset == null) throw new NullPointerException();
	        return StringCoding.encode(charset, value, 0, value.length);
	    }
		
	   //使用平台的默认字符集将此 String编码为字节序列，将结果存储到新的字节数组中。	
		//其中用到了StringCoding类 
	    public byte[] getBytes() {
	        return StringCoding.encode(value, 0, value.length);
	    }
	
	  	//String 重写equals方法
		//1.判断两个对象是否== 如果相等返回true
		//2.判断比较的另一个对象类型是否为String 如果否 返回false,如果是 ，进入3
		//3.将这个对象转型成String,比较两个String对象的长度是否相同 如果否 返回false,如果是 进入4
		//4.遍历每一个char比较是否相同 如果是 比较下一个 如果否 返回false
	    public boolean equals(Object anObject) {
	        if (this == anObject) {
	            return true;
	        }
	        if (anObject instanceof String) {
	            String anotherString = (String)anObject;
	            int n = value.length;
	            if (n == anotherString.value.length) {
	                char v1[] = value;
	                char v2[] = anotherString.value;
	                int i = 0;
	                while (n-- != 0) {
	                    if (v1[i] != v2[i])
	                        return false;
	                    i++;
	                }
	                return true;
	            }
	        }
	        return false;
	    }
	
	  	//contentEquals只要求另一个对象是CharSequence或其子类的对象 判断内容是否相同
	    public boolean contentEquals(StringBuffer sb) {
	        return contentEquals((CharSequence)sb);
	    }
	
	    private boolean nonSyncContentEquals(AbstractStringBuilder sb) {
	        char v1[] = value;
	        char v2[] = sb.getValue();
	        int n = v1.length;
	        if (n != sb.length()) {
	            return false;
	        }
	        for (int i = 0; i < n; i++) {
	            if (v1[i] != v2[i]) {
	                return false;
	            }
	        }
	        return true;
	    }
	
	   
	    public boolean contentEquals(CharSequence cs) {
	        // Argument is a StringBuffer, StringBuilder
	        if (cs instanceof AbstractStringBuilder) {
	            if (cs instanceof StringBuffer) {
	                synchronized(cs) {
	                   return nonSyncContentEquals((AbstractStringBuilder)cs);
	                }
	            } else {
	                return nonSyncContentEquals((AbstractStringBuilder)cs);
	            }
	        }
	        // Argument is a String
	        if (cs instanceof String) {
	            return equals(cs);
	        }
	        // Argument is a generic CharSequence
	        char v1[] = value;
	        int n = v1.length;
	        if (n != cs.length()) {
	            return false;
	        }
	        for (int i = 0; i < n; i++) {
	            if (v1[i] != cs.charAt(i)) {
	                return false;
	            }
	        }
	        return true;
	    }
	
	   
	    public boolean equalsIgnoreCase(String anotherString) {
	        return (this == anotherString) ? true
	                : (anotherString != null)
	                && (anotherString.value.length == value.length)
	                && regionMatches(true, 0, anotherString, 0, value.length);
	    }
	

	    //比较两个String对象大小 
		//1.从两个字符串找到最小字符串的长度lim
		//2.从索引0位置开始 到 lim-1 判断两个字符串索引位置字符的大小比较 哪个大就大， 如果相等 继续索引位置比较大小 
		//3.如果前lim都是相同的，比较两个String长度大 的 大 
	    public int compareTo(String anotherString) {
	        int len1 = value.length;
	        int len2 = anotherString.value.length;
	        int lim = Math.min(len1, len2);
	        char v1[] = value;
	        char v2[] = anotherString.value;
	
	        int k = 0;
	        while (k < lim) {
	            char c1 = v1[k];
	            char c2 = v2[k];
	            if (c1 != c2) {
	                return c1 - c2;
	            }
	            k++;
	        }
	        return len1 - len2;
	    }
		
		//最后，这个CASE_INSENSITIVE_ORDER在下面内部类中会说到，其根本就是持有一个静态内部类，用于忽略大小写得比较两个字符串。
	    public static final Comparator<String> CASE_INSENSITIVE_ORDER
	                                         = new CaseInsensitiveComparator();
		//忽略大小写比较
	    private static class CaseInsensitiveComparator
	            implements Comparator<String>, java.io.Serializable {
	        // use serialVersionUID from JDK 1.2.2 for interoperability
	        private static final long serialVersionUID = 8575799808933029326L;
			
			
	        public int compare(String s1, String s2) {
	            int n1 = s1.length();
	            int n2 = s2.length();
	            int min = Math.min(n1, n2);
	            for (int i = 0; i < min; i++) {
	                char c1 = s1.charAt(i);
	                char c2 = s2.charAt(i);
	                if (c1 != c2) {
	                    c1 = Character.toUpperCase(c1);
	                    c2 = Character.toUpperCase(c2);
	                    if (c1 != c2) {
	                        c1 = Character.toLowerCase(c1);
	                        c2 = Character.toLowerCase(c2);
	                        if (c1 != c2) {
	                            // No overflow because of numeric promotion
	                            return c1 - c2;
	                        }
	                    }
	                }
	            }
	            return n1 - n2;
	        }
	
	        private Object readResolve() { return CASE_INSENSITIVE_ORDER; }
	    }
	
	  
	    public int compareToIgnoreCase(String str) {
	        return CASE_INSENSITIVE_ORDER.compare(this, str);
	    }
	
	 
	    public boolean regionMatches(int toffset, String other, int ooffset,
	            int len) {
	        char ta[] = value;
	        int to = toffset;
	        char pa[] = other.value;
	        int po = ooffset;
	        // Note: toffset, ooffset, or len might be near -1>>>1.
	        if ((ooffset < 0) || (toffset < 0)
	                || (toffset > (long)value.length - len)
	                || (ooffset > (long)other.value.length - len)) {
	            return false;
	        }
	        while (len-- > 0) {
	            if (ta[to++] != pa[po++]) {
	                return false;
	            }
	        }
	        return true;
	    }
	
	   
	    public boolean regionMatches(boolean ignoreCase, int toffset,
	            String other, int ooffset, int len) {
	        char ta[] = value;
	        int to = toffset;
	        char pa[] = other.value;
	        int po = ooffset;
	        // Note: toffset, ooffset, or len might be near -1>>>1.
	        if ((ooffset < 0) || (toffset < 0)
	                || (toffset > (long)value.length - len)
	                || (ooffset > (long)other.value.length - len)) {
	            return false;
	        }
	        while (len-- > 0) {
	            char c1 = ta[to++];
	            char c2 = pa[po++];
	            if (c1 == c2) {
	                continue;
	            }
	            if (ignoreCase) {
	                // If characters don't match but case may be ignored,
	                // try converting both characters to uppercase.
	                // If the results match, then the comparison scan should
	                // continue.
	                char u1 = Character.toUpperCase(c1);
	                char u2 = Character.toUpperCase(c2);
	                if (u1 == u2) {
	                    continue;
	                }
	                // Unfortunately, conversion to uppercase does not work properly
	                // for the Georgian alphabet, which has strange rules about case
	                // conversion.  So we need to make one last check before
	                // exiting.
	                if (Character.toLowerCase(u1) == Character.toLowerCase(u2)) {
	                    continue;
	                }
	            }
	            return false;
	        }
	        return true;
	    }
	
	    public boolean startsWith(String prefix, int toffset) {
	        char ta[] = value;
	        int to = toffset;
	        char pa[] = prefix.value;
	        int po = 0;
	        int pc = prefix.value.length;
	        // Note: toffset might be near -1>>>1.
	        if ((toffset < 0) || (toffset > value.length - pc)) {
	            return false;
	        }
	        while (--pc >= 0) {
	            if (ta[to++] != pa[po++]) {
	                return false;
	            }
	        }
	        return true;
	    }
	
	   
	    public boolean startsWith(String prefix) {
	        return startsWith(prefix, 0);
	    }
	
	   
	    public boolean endsWith(String suffix) {
	        return startsWith(suffix, value.length - suffix.value.length);
	    }
	
	   
	    public int hashCode() {
	        int h = hash;
	        if (h == 0 && value.length > 0) {
	            char val[] = value;
	
	            for (int i = 0; i < value.length; i++) {
	                h = 31 * h + val[i];
	            }
	            hash = h;
	        }
	        return h;
	    }
	
	    
	    public int indexOf(int ch) {
	        return indexOf(ch, 0);
	    }
	
	   
	    public int indexOf(int ch, int fromIndex) {
	        final int max = value.length;
	        if (fromIndex < 0) {
	            fromIndex = 0;
	        } else if (fromIndex >= max) {
	            // Note: fromIndex might be near -1>>>1.
	            return -1;
	        }
	
	        if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
	            // handle most cases here (ch is a BMP code point or a
	            // negative value (invalid code point))
	            final char[] value = this.value;
	            for (int i = fromIndex; i < max; i++) {
	                if (value[i] == ch) {
	                    return i;
	                }
	            }
	            return -1;
	        } else {
	            return indexOfSupplementary(ch, fromIndex);
	        }
	    }
	
	    /**
	     * Handles (rare) calls of indexOf with a supplementary character.
	     */
	    private int indexOfSupplementary(int ch, int fromIndex) {
	        if (Character.isValidCodePoint(ch)) {
	            final char[] value = this.value;
	            final char hi = Character.highSurrogate(ch);
	            final char lo = Character.lowSurrogate(ch);
	            final int max = value.length - 1;
	            for (int i = fromIndex; i < max; i++) {
	                if (value[i] == hi && value[i + 1] == lo) {
	                    return i;
	                }
	            }
	        }
	        return -1;
	    }
	
	  
	    public int lastIndexOf(int ch) {
	        return lastIndexOf(ch, value.length - 1);
	    }
	
	   
	    public int lastIndexOf(int ch, int fromIndex) {
	        if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
	            // handle most cases here (ch is a BMP code point or a
	            // negative value (invalid code point))
	            final char[] value = this.value;
	            int i = Math.min(fromIndex, value.length - 1);
	            for (; i >= 0; i--) {
	                if (value[i] == ch) {
	                    return i;
	                }
	            }
	            return -1;
	        } else {
	            return lastIndexOfSupplementary(ch, fromIndex);
	        }
	    }
	
	    /**
	     * Handles (rare) calls of lastIndexOf with a supplementary character.
	     */
	    private int lastIndexOfSupplementary(int ch, int fromIndex) {
	        if (Character.isValidCodePoint(ch)) {
	            final char[] value = this.value;
	            char hi = Character.highSurrogate(ch);
	            char lo = Character.lowSurrogate(ch);
	            int i = Math.min(fromIndex, value.length - 2);
	            for (; i >= 0; i--) {
	                if (value[i] == hi && value[i + 1] == lo) {
	                    return i;
	                }
	            }
	        }
	        return -1;
	    }
	
	  
	    public int indexOf(String str) {
	        return indexOf(str, 0);
	    }
	
	  
	    public int indexOf(String str, int fromIndex) {
	        return indexOf(value, 0, value.length,
	                str.value, 0, str.value.length, fromIndex);
	    }
	
	  
	    static int indexOf(char[] source, int sourceOffset, int sourceCount,
	            String target, int fromIndex) {
	        return indexOf(source, sourceOffset, sourceCount,
	                       target.value, 0, target.value.length,
	                       fromIndex);
	    }
	
	   
	    static int indexOf(char[] source, int sourceOffset, int sourceCount,
	            char[] target, int targetOffset, int targetCount,
	            int fromIndex) {
	        if (fromIndex >= sourceCount) {
	            return (targetCount == 0 ? sourceCount : -1);
	        }
	        if (fromIndex < 0) {
	            fromIndex = 0;
	        }
	        if (targetCount == 0) {
	            return fromIndex;
	        }
	
	        char first = target[targetOffset];
	        int max = sourceOffset + (sourceCount - targetCount);
	
	        for (int i = sourceOffset + fromIndex; i <= max; i++) {
	            /* Look for first character. */
	            if (source[i] != first) {
	                while (++i <= max && source[i] != first);
	            }
	
	            /* Found first character, now look at the rest of v2 */
	            if (i <= max) {
	                int j = i + 1;
	                int end = j + targetCount - 1;
	                for (int k = targetOffset + 1; j < end && source[j]
	                        == target[k]; j++, k++);
	
	                if (j == end) {
	                    /* Found whole string. */
	                    return i - sourceOffset;
	                }
	            }
	        }
	        return -1;
	    }
	
	   
	    public int lastIndexOf(String str) {
	        return lastIndexOf(str, value.length);
	    }
	
	   
	    public int lastIndexOf(String str, int fromIndex) {
	        return lastIndexOf(value, 0, value.length,
	                str.value, 0, str.value.length, fromIndex);
	    }
	
	  
	    static int lastIndexOf(char[] source, int sourceOffset, int sourceCount,
	            String target, int fromIndex) {
	        return lastIndexOf(source, sourceOffset, sourceCount,
	                       target.value, 0, target.value.length,
	                       fromIndex);
	    }
	
	   
	    static int lastIndexOf(char[] source, int sourceOffset, int sourceCount,
	            char[] target, int targetOffset, int targetCount,
	            int fromIndex) {
	        /*
	         * Check arguments; return immediately where possible. For
	         * consistency, don't check for null str.
	         */
	        int rightIndex = sourceCount - targetCount;
	        if (fromIndex < 0) {
	            return -1;
	        }
	        if (fromIndex > rightIndex) {
	            fromIndex = rightIndex;
	        }
	        /* Empty string always matches. */
	        if (targetCount == 0) {
	            return fromIndex;
	        }
	
	        int strLastIndex = targetOffset + targetCount - 1;
	        char strLastChar = target[strLastIndex];
	        int min = sourceOffset + targetCount - 1;
	        int i = min + fromIndex;
	
	    startSearchForLastChar:
	        while (true) {
	            while (i >= min && source[i] != strLastChar) {
	                i--;
	            }
	            if (i < min) {
	                return -1;
	            }
	            int j = i - 1;
	            int start = j - (targetCount - 1);
	            int k = strLastIndex - 1;
	
	            while (j > start) {
	                if (source[j--] != target[k--]) {
	                    i--;
	                    continue startSearchForLastChar;
	                }
	            }
	            return start - sourceOffset + 1;
	        }
	    }
	
	  
	    public String substring(int beginIndex) {
	        if (beginIndex < 0) {
	            throw new StringIndexOutOfBoundsException(beginIndex);
	        }
	        int subLen = value.length - beginIndex;
	        if (subLen < 0) {
	            throw new StringIndexOutOfBoundsException(subLen);
	        }
	        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
	    }
	
	   
	    public String substring(int beginIndex, int endIndex) {
	        if (beginIndex < 0) {
	            throw new StringIndexOutOfBoundsException(beginIndex);
	        }
	        if (endIndex > value.length) {
	            throw new StringIndexOutOfBoundsException(endIndex);
	        }
	        int subLen = endIndex - beginIndex;
	        if (subLen < 0) {
	            throw new StringIndexOutOfBoundsException(subLen);
	        }
	        return ((beginIndex == 0) && (endIndex == value.length)) ? this
	                : new String(value, beginIndex, subLen);
	    }
	
	   
	    public CharSequence subSequence(int beginIndex, int endIndex) {
	        return this.substring(beginIndex, endIndex);
	    }
	
	   
	    public String concat(String str) {
	        int otherLen = str.length();
	        if (otherLen == 0) {
	            return this;
	        }
	        int len = value.length;
	        char buf[] = Arrays.copyOf(value, len + otherLen);
	        str.getChars(buf, len);
	        return new String(buf, true);
	    }
	
	   
	    public String replace(char oldChar, char newChar) {
	        if (oldChar != newChar) {
	            int len = value.length;
	            int i = -1;
	            char[] val = value; /* avoid getfield opcode */
	
	            while (++i < len) {
	                if (val[i] == oldChar) {
	                    break;
	                }
	            }
	            if (i < len) {
	                char buf[] = new char[len];
	                for (int j = 0; j < i; j++) {
	                    buf[j] = val[j];
	                }
	                while (i < len) {
	                    char c = val[i];
	                    buf[i] = (c == oldChar) ? newChar : c;
	                    i++;
	                }
	                return new String(buf, true);
	            }
	        }
	        return this;
	    }
	
	   
	    public boolean matches(String regex) {
	        return Pattern.matches(regex, this);
	    }
	
	    /**
	     * Returns true if and only if this string contains the specified
	     * sequence of char values.
	     *
	     * @param s the sequence to search for
	     * @return true if this string contains {@code s}, false otherwise
	     * @since 1.5
	     */
	    public boolean contains(CharSequence s) {
	        return indexOf(s.toString()) > -1;
	    }
	
	   
	    public String replaceFirst(String regex, String replacement) {
	        return Pattern.compile(regex).matcher(this).replaceFirst(replacement);
	    }
	
	    
	    public String replaceAll(String regex, String replacement) {
	        return Pattern.compile(regex).matcher(this).replaceAll(replacement);
	    }
	
	    /**
	     * Replaces each substring of this string that matches the literal target
	     * sequence with the specified literal replacement sequence. The
	     * replacement proceeds from the beginning of the string to the end, for
	     * example, replacing "aa" with "b" in the string "aaa" will result in
	     * "ba" rather than "ab".
	     *
	     * @param  target The sequence of char values to be replaced
	     * @param  replacement The replacement sequence of char values
	     * @return  The resulting string
	     * @since 1.5
	     */
	    public String replace(CharSequence target, CharSequence replacement) {
	        return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(
	                this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
	    }
	
	   
	    public String[] split(String regex, int limit) {
	        /* fastpath if the regex is a
	         (1)one-char String and this character is not one of the
	            RegEx's meta characters ".$|()[{^?*+\\", or
	         (2)two-char String and the first char is the backslash and
	            the second is not the ascii digit or ascii letter.
	         */
	        char ch = 0;
	        if (((regex.value.length == 1 &&
	             ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
	             (regex.length() == 2 &&
	              regex.charAt(0) == '\\' &&
	              (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
	              ((ch-'a')|('z'-ch)) < 0 &&
	              ((ch-'A')|('Z'-ch)) < 0)) &&
	            (ch < Character.MIN_HIGH_SURROGATE ||
	             ch > Character.MAX_LOW_SURROGATE))
	        {
	            int off = 0;
	            int next = 0;
	            boolean limited = limit > 0;
	            ArrayList<String> list = new ArrayList<>();
	            while ((next = indexOf(ch, off)) != -1) {
	                if (!limited || list.size() < limit - 1) {
	                    list.add(substring(off, next));
	                    off = next + 1;
	                } else {    // last one
	                    //assert (list.size() == limit - 1);
	                    list.add(substring(off, value.length));
	                    off = value.length;
	                    break;
	                }
	            }
	            // If no match was found, return this
	            if (off == 0)
	                return new String[]{this};
	
	            // Add remaining segment
	            if (!limited || list.size() < limit)
	                list.add(substring(off, value.length));
	
	            // Construct result
	            int resultSize = list.size();
	            if (limit == 0) {
	                while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
	                    resultSize--;
	                }
	            }
	            String[] result = new String[resultSize];
	            return list.subList(0, resultSize).toArray(result);
	        }
	        return Pattern.compile(regex).split(this, limit);
	    }
	
	   
	    public String[] split(String regex) {
	        return split(regex, 0);
	    }
	
	   
	    public static String join(CharSequence delimiter, CharSequence... elements) {
	        Objects.requireNonNull(delimiter);
	        Objects.requireNonNull(elements);
	        // Number of elements not likely worth Arrays.stream overhead.
	        StringJoiner joiner = new StringJoiner(delimiter);
	        for (CharSequence cs: elements) {
	            joiner.add(cs);
	        }
	        return joiner.toString();
	    }
	
	   
	    public static String join(CharSequence delimiter,
	            Iterable<? extends CharSequence> elements) {
	        Objects.requireNonNull(delimiter);
	        Objects.requireNonNull(elements);
	        StringJoiner joiner = new StringJoiner(delimiter);
	        for (CharSequence cs: elements) {
	            joiner.add(cs);
	        }
	        return joiner.toString();
	    }
	
	   
	    public String toLowerCase(Locale locale) {
	        if (locale == null) {
	            throw new NullPointerException();
	        }
	
	        int firstUpper;
	        final int len = value.length;
	
	        /* Now check if there are any characters that need to be changed. */
	        scan: {
	            for (firstUpper = 0 ; firstUpper < len; ) {
	                char c = value[firstUpper];
	                if ((c >= Character.MIN_HIGH_SURROGATE)
	                        && (c <= Character.MAX_HIGH_SURROGATE)) {
	                    int supplChar = codePointAt(firstUpper);
	                    if (supplChar != Character.toLowerCase(supplChar)) {
	                        break scan;
	                    }
	                    firstUpper += Character.charCount(supplChar);
	                } else {
	                    if (c != Character.toLowerCase(c)) {
	                        break scan;
	                    }
	                    firstUpper++;
	                }
	            }
	            return this;
	        }
	
	        char[] result = new char[len];
	        int resultOffset = 0;  /* result may grow, so i+resultOffset
	                                * is the write location in result */
	
	        /* Just copy the first few lowerCase characters. */
	        System.arraycopy(value, 0, result, 0, firstUpper);
	
	        String lang = locale.getLanguage();
	        boolean localeDependent =
	                (lang == "tr" || lang == "az" || lang == "lt");
	        char[] lowerCharArray;
	        int lowerChar;
	        int srcChar;
	        int srcCount;
	        for (int i = firstUpper; i < len; i += srcCount) {
	            srcChar = (int)value[i];
	            if ((char)srcChar >= Character.MIN_HIGH_SURROGATE
	                    && (char)srcChar <= Character.MAX_HIGH_SURROGATE) {
	                srcChar = codePointAt(i);
	                srcCount = Character.charCount(srcChar);
	            } else {
	                srcCount = 1;
	            }
	            if (localeDependent ||
	                srcChar == '\u03A3' || // GREEK CAPITAL LETTER SIGMA
	                srcChar == '\u0130') { // LATIN CAPITAL LETTER I WITH DOT ABOVE
	                lowerChar = ConditionalSpecialCasing.toLowerCaseEx(this, i, locale);
	            } else {
	                lowerChar = Character.toLowerCase(srcChar);
	            }
	            if ((lowerChar == Character.ERROR)
	                    || (lowerChar >= Character.MIN_SUPPLEMENTARY_CODE_POINT)) {
	                if (lowerChar == Character.ERROR) {
	                    lowerCharArray =
	                            ConditionalSpecialCasing.toLowerCaseCharArray(this, i, locale);
	                } else if (srcCount == 2) {
	                    resultOffset += Character.toChars(lowerChar, result, i + resultOffset) - srcCount;
	                    continue;
	                } else {
	                    lowerCharArray = Character.toChars(lowerChar);
	                }
	
	                /* Grow result if needed */
	                int mapLen = lowerCharArray.length;
	                if (mapLen > srcCount) {
	                    char[] result2 = new char[result.length + mapLen - srcCount];
	                    System.arraycopy(result, 0, result2, 0, i + resultOffset);
	                    result = result2;
	                }
	                for (int x = 0; x < mapLen; ++x) {
	                    result[i + resultOffset + x] = lowerCharArray[x];
	                }
	                resultOffset += (mapLen - srcCount);
	            } else {
	                result[i + resultOffset] = (char)lowerChar;
	            }
	        }
	        return new String(result, 0, len + resultOffset);
	    }
	
	   
	    public String toLowerCase() {
	        return toLowerCase(Locale.getDefault());
	    }
	
	   
	    public String toUpperCase(Locale locale) {
	        if (locale == null) {
	            throw new NullPointerException();
	        }
	
	        int firstLower;
	        final int len = value.length;
	
	        /* Now check if there are any characters that need to be changed. */
	        scan: {
	            for (firstLower = 0 ; firstLower < len; ) {
	                int c = (int)value[firstLower];
	                int srcCount;
	                if ((c >= Character.MIN_HIGH_SURROGATE)
	                        && (c <= Character.MAX_HIGH_SURROGATE)) {
	                    c = codePointAt(firstLower);
	                    srcCount = Character.charCount(c);
	                } else {
	                    srcCount = 1;
	                }
	                int upperCaseChar = Character.toUpperCaseEx(c);
	                if ((upperCaseChar == Character.ERROR)
	                        || (c != upperCaseChar)) {
	                    break scan;
	                }
	                firstLower += srcCount;
	            }
	            return this;
	        }
	
	        /* result may grow, so i+resultOffset is the write location in result */
	        int resultOffset = 0;
	        char[] result = new char[len]; /* may grow */
	
	        /* Just copy the first few upperCase characters. */
	        System.arraycopy(value, 0, result, 0, firstLower);
	
	        String lang = locale.getLanguage();
	        boolean localeDependent =
	                (lang == "tr" || lang == "az" || lang == "lt");
	        char[] upperCharArray;
	        int upperChar;
	        int srcChar;
	        int srcCount;
	        for (int i = firstLower; i < len; i += srcCount) {
	            srcChar = (int)value[i];
	            if ((char)srcChar >= Character.MIN_HIGH_SURROGATE &&
	                (char)srcChar <= Character.MAX_HIGH_SURROGATE) {
	                srcChar = codePointAt(i);
	                srcCount = Character.charCount(srcChar);
	            } else {
	                srcCount = 1;
	            }
	            if (localeDependent) {
	                upperChar = ConditionalSpecialCasing.toUpperCaseEx(this, i, locale);
	            } else {
	                upperChar = Character.toUpperCaseEx(srcChar);
	            }
	            if ((upperChar == Character.ERROR)
	                    || (upperChar >= Character.MIN_SUPPLEMENTARY_CODE_POINT)) {
	                if (upperChar == Character.ERROR) {
	                    if (localeDependent) {
	                        upperCharArray =
	                                ConditionalSpecialCasing.toUpperCaseCharArray(this, i, locale);
	                    } else {
	                        upperCharArray = Character.toUpperCaseCharArray(srcChar);
	                    }
	                } else if (srcCount == 2) {
	                    resultOffset += Character.toChars(upperChar, result, i + resultOffset) - srcCount;
	                    continue;
	                } else {
	                    upperCharArray = Character.toChars(upperChar);
	                }
	
	                /* Grow result if needed */
	                int mapLen = upperCharArray.length;
	                if (mapLen > srcCount) {
	                    char[] result2 = new char[result.length + mapLen - srcCount];
	                    System.arraycopy(result, 0, result2, 0, i + resultOffset);
	                    result = result2;
	                }
	                for (int x = 0; x < mapLen; ++x) {
	                    result[i + resultOffset + x] = upperCharArray[x];
	                }
	                resultOffset += (mapLen - srcCount);
	            } else {
	                result[i + resultOffset] = (char)upperChar;
	            }
	        }
	        return new String(result, 0, len + resultOffset);
	    }
	
	 
	    public String toUpperCase() {
	        return toUpperCase(Locale.getDefault());
	    }
	
	   
	    public String trim() {
	        int len = value.length;
	        int st = 0;
	        char[] val = value;    /* avoid getfield opcode */
	
	        while ((st < len) && (val[st] <= ' ')) {
	            st++;
	        }
	        while ((st < len) && (val[len - 1] <= ' ')) {
	            len--;
	        }
	        return ((st > 0) || (len < value.length)) ? substring(st, len) : this;
	    }
	
	    /**
	     * This object (which is already a string!) is itself returned.
	     *
	     * @return  the string itself.
	     */
	    public String toString() {
	        return this;
	    }
	
	   
	    public char[] toCharArray() {
	        // Cannot use Arrays.copyOf because of class initialization order issues
	        char result[] = new char[value.length];
	        System.arraycopy(value, 0, result, 0, value.length);
	        return result;
	    }
	
	   //比如 str=String.format("Hi,%s", "小超");   格式化
	    public static String format(String format, Object... args) {
	        return new Formatter().format(format, args).toString();
	    }
	
	   
	    public static String format(Locale l, String format, Object... args) {
	        return new Formatter(l).format(format, args).toString();
	    }
	
	    public static String valueOf(Object obj) {
	        return (obj == null) ? "null" : obj.toString();
	    }
	
	   
	    public static String valueOf(char data[]) {
	        return new String(data);
	    }
	
	  
	    public static String valueOf(char data[], int offset, int count) {
	        return new String(data, offset, count);
	    }
	
	    
	    public static String copyValueOf(char data[], int offset, int count) {
	        return new String(data, offset, count);
	    }
	
	    /**
	     * Equivalent to {@link #valueOf(char[])}.
	     *
	     * @param   data   the character array.
	     * @return  a {@code String} that contains the characters of the
	     *          character array.
	     */
	    public static String copyValueOf(char data[]) {
	        return new String(data);
	    }
	
	  
	    public static String valueOf(boolean b) {
	        return b ? "true" : "false";
	    }
	
	  
	    public static String valueOf(char c) {
	        char data[] = {c};
	        return new String(data, true);
	    }
	
	  
	    public static String valueOf(int i) {
	        return Integer.toString(i);
	    }
	
	   
	    public static String valueOf(long l) {
	        return Long.toString(l);
	    }
	
	    public static String valueOf(float f) {
	        return Float.toString(f);
	    }
	
	    
	    public static String valueOf(double d) {
	        return Double.toString(d);
	    }
	
	   
	    public native String intern();
	}


##另附StringBuffer StringBuilder 源码

#StringBuffer源码


	package java.lang;

	import java.util.Arrays;
	
	 public final class StringBuffer
	    extends AbstractStringBuilder
	    implements java.io.Serializable, CharSequence
	{

    /**
     * A cache of the last value returned by toString. Cleared
     * whenever the StringBuffer is modified.
     */
    private transient char[] toStringCache;

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    static final long serialVersionUID = 3388685877147921107L;

    /**
     * Constructs a string buffer with no characters in it and an
     * initial capacity of 16 characters.
     */
    public StringBuffer() {
        super(16);
    }

    /**
     * Constructs a string buffer with no characters in it and
     * the specified initial capacity.
     *
     * @param      capacity  the initial capacity.
     * @exception  NegativeArraySizeException  if the {@code capacity}
     *               argument is less than {@code 0}.
     */
    public StringBuffer(int capacity) {
        super(capacity);
    }

    /**
     * Constructs a string buffer initialized to the contents of the
     * specified string. The initial capacity of the string buffer is
     * {@code 16} plus the length of the string argument.
     *
     * @param   str   the initial contents of the buffer.
     */
    public StringBuffer(String str) {
        super(str.length() + 16);
        append(str);
    }

    /**
     * Constructs a string buffer that contains the same characters
     * as the specified {@code CharSequence}. The initial capacity of
     * the string buffer is {@code 16} plus the length of the
     * {@code CharSequence} argument.
     * <p>
     * If the length of the specified {@code CharSequence} is
     * less than or equal to zero, then an empty buffer of capacity
     * {@code 16} is returned.
     *
     * @param      seq   the sequence to copy.
     * @since 1.5
     */
    public StringBuffer(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }

    @Override
    public synchronized int length() {
        return count;
    }

    @Override
    public synchronized int capacity() {
        return value.length;
    }


    @Override
    public synchronized void ensureCapacity(int minimumCapacity) {
        super.ensureCapacity(minimumCapacity);
    }

    /**
     * @since      1.5
     */
    @Override
    public synchronized void trimToSize() {
        super.trimToSize();
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @see        #length()
     */
    @Override
    public synchronized void setLength(int newLength) {
        toStringCache = null;
        super.setLength(newLength);
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @see        #length()
     */
    @Override
    public synchronized char charAt(int index) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        return value[index];
    }

    /**
     * @since      1.5
     */
    @Override
    public synchronized int codePointAt(int index) {
        return super.codePointAt(index);
    }

    /**
     * @since     1.5
     */
    @Override
    public synchronized int codePointBefore(int index) {
        return super.codePointBefore(index);
    }

    /**
     * @since     1.5
     */
    @Override
    public synchronized int codePointCount(int beginIndex, int endIndex) {
        return super.codePointCount(beginIndex, endIndex);
    }

    /**
     * @since     1.5
     */
    @Override
    public synchronized int offsetByCodePoints(int index, int codePointOffset) {
        return super.offsetByCodePoints(index, codePointOffset);
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public synchronized void getChars(int srcBegin, int srcEnd, char[] dst,
                                      int dstBegin)
    {
        super.getChars(srcBegin, srcEnd, dst, dstBegin);
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @see        #length()
     */
    @Override
    public synchronized void setCharAt(int index, char ch) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        toStringCache = null;
        value[index] = ch;
    }

    @Override
    public synchronized StringBuffer append(Object obj) {
        toStringCache = null;
        super.append(String.valueOf(obj));
        return this;
    }

    @Override
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }

    /**
     * Appends the specified {@code StringBuffer} to this sequence.
     * <p>
     * The characters of the {@code StringBuffer} argument are appended,
     * in order, to the contents of this {@code StringBuffer}, increasing the
     * length of this {@code StringBuffer} by the length of the argument.
     * If {@code sb} is {@code null}, then the four characters
     * {@code "null"} are appended to this {@code StringBuffer}.
     * <p>
     * Let <i>n</i> be the length of the old character sequence, the one
     * contained in the {@code StringBuffer} just prior to execution of the
     * {@code append} method. Then the character at index <i>k</i> in
     * the new character sequence is equal to the character at index <i>k</i>
     * in the old character sequence, if <i>k</i> is less than <i>n</i>;
     * otherwise, it is equal to the character at index <i>k-n</i> in the
     * argument {@code sb}.
     * <p>
     * This method synchronizes on {@code this}, the destination
     * object, but does not synchronize on the source ({@code sb}).
     *
     * @param   sb   the {@code StringBuffer} to append.
     * @return  a reference to this object.
     * @since 1.4
     */
    public synchronized StringBuffer append(StringBuffer sb) {
        toStringCache = null;
        super.append(sb);
        return this;
    }

    /**
     * @since 1.8
     */
    @Override
    synchronized StringBuffer append(AbstractStringBuilder asb) {
        toStringCache = null;
        super.append(asb);
        return this;
    }

    /**
     * Appends the specified {@code CharSequence} to this
     * sequence.
     * <p>
     * The characters of the {@code CharSequence} argument are appended,
     * in order, increasing the length of this sequence by the length of the
     * argument.
     *
     * <p>The result of this method is exactly the same as if it were an
     * invocation of this.append(s, 0, s.length());
     *
     * <p>This method synchronizes on {@code this}, the destination
     * object, but does not synchronize on the source ({@code s}).
     *
     * <p>If {@code s} is {@code null}, then the four characters
     * {@code "null"} are appended.
     *
     * @param   s the {@code CharSequence} to append.
     * @return  a reference to this object.
     * @since 1.5
     */
    @Override
    public synchronized StringBuffer append(CharSequence s) {
        toStringCache = null;
        super.append(s);
        return this;
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @since      1.5
     */
    @Override
    public synchronized StringBuffer append(CharSequence s, int start, int end)
    {
        toStringCache = null;
        super.append(s, start, end);
        return this;
    }

    @Override
    public synchronized StringBuffer append(char[] str) {
        toStringCache = null;
        super.append(str);
        return this;
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public synchronized StringBuffer append(char[] str, int offset, int len) {
        toStringCache = null;
        super.append(str, offset, len);
        return this;
    }

    @Override
    public synchronized StringBuffer append(boolean b) {
        toStringCache = null;
        super.append(b);
        return this;
    }

    @Override
    public synchronized StringBuffer append(char c) {
        toStringCache = null;
        super.append(c);
        return this;
    }

    @Override
    public synchronized StringBuffer append(int i) {
        toStringCache = null;
        super.append(i);
        return this;
    }

    /**
     * @since 1.5
     */
    @Override
    public synchronized StringBuffer appendCodePoint(int codePoint) {
        toStringCache = null;
        super.appendCodePoint(codePoint);
        return this;
    }

    @Override
    public synchronized StringBuffer append(long lng) {
        toStringCache = null;
        super.append(lng);
        return this;
    }

    @Override
    public synchronized StringBuffer append(float f) {
        toStringCache = null;
        super.append(f);
        return this;
    }

    @Override
    public synchronized StringBuffer append(double d) {
        toStringCache = null;
        super.append(d);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     * @since      1.2
     */
    @Override
    public synchronized StringBuffer delete(int start, int end) {
        toStringCache = null;
        super.delete(start, end);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     * @since      1.2
     */
    @Override
    public synchronized StringBuffer deleteCharAt(int index) {
        toStringCache = null;
        super.deleteCharAt(index);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     * @since      1.2
     */
    @Override
    public synchronized StringBuffer replace(int start, int end, String str) {
        toStringCache = null;
        super.replace(start, end, str);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     * @since      1.2
     */
    @Override
    public synchronized String substring(int start) {
        return substring(start, count);
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @since      1.4
     */
    @Override
    public synchronized CharSequence subSequence(int start, int end) {
        return super.substring(start, end);
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     * @since      1.2
     */
    @Override
    public synchronized String substring(int start, int end) {
        return super.substring(start, end);
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     * @since      1.2
     */
    @Override
    public synchronized StringBuffer insert(int index, char[] str, int offset,
                                            int len)
    {
        toStringCache = null;
        super.insert(index, str, offset, len);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public synchronized StringBuffer insert(int offset, Object obj) {
        toStringCache = null;
        super.insert(offset, String.valueOf(obj));
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public synchronized StringBuffer insert(int offset, String str) {
        toStringCache = null;
        super.insert(offset, str);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public synchronized StringBuffer insert(int offset, char[] str) {
        toStringCache = null;
        super.insert(offset, str);
        return this;
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @since      1.5
     */
    @Override
    public StringBuffer insert(int dstOffset, CharSequence s) {
        // Note, synchronization achieved via invocations of other StringBuffer methods
        // after narrowing of s to specific type
        // Ditto for toStringCache clearing
        super.insert(dstOffset, s);
        return this;
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * @since      1.5
     */
    @Override
    public synchronized StringBuffer insert(int dstOffset, CharSequence s,
            int start, int end)
    {
        toStringCache = null;
        super.insert(dstOffset, s, start, end);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public  StringBuffer insert(int offset, boolean b) {
        // Note, synchronization achieved via invocation of StringBuffer insert(int, String)
        // after conversion of b to String by super class method
        // Ditto for toStringCache clearing
        super.insert(offset, b);
        return this;
    }

    /**
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public synchronized StringBuffer insert(int offset, char c) {
        toStringCache = null;
        super.insert(offset, c);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public StringBuffer insert(int offset, int i) {
        // Note, synchronization achieved via invocation of StringBuffer insert(int, String)
        // after conversion of i to String by super class method
        // Ditto for toStringCache clearing
        super.insert(offset, i);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public StringBuffer insert(int offset, long l) {
        // Note, synchronization achieved via invocation of StringBuffer insert(int, String)
        // after conversion of l to String by super class method
        // Ditto for toStringCache clearing
        super.insert(offset, l);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public StringBuffer insert(int offset, float f) {
        // Note, synchronization achieved via invocation of StringBuffer insert(int, String)
        // after conversion of f to String by super class method
        // Ditto for toStringCache clearing
        super.insert(offset, f);
        return this;
    }

    /**
     * @throws StringIndexOutOfBoundsException {@inheritDoc}
     */
    @Override
    public StringBuffer insert(int offset, double d) {
        // Note, synchronization achieved via invocation of StringBuffer insert(int, String)
        // after conversion of d to String by super class method
        // Ditto for toStringCache clearing
        super.insert(offset, d);
        return this;
    }

    /**
     * @since      1.4
     */
    @Override
    public int indexOf(String str) {
        // Note, synchronization achieved via invocations of other StringBuffer methods
        return super.indexOf(str);
    }

    /**
     * @since      1.4
     */
    @Override
    public synchronized int indexOf(String str, int fromIndex) {
        return super.indexOf(str, fromIndex);
    }

    /**
     * @since      1.4
     */
    @Override
    public int lastIndexOf(String str) {
        // Note, synchronization achieved via invocations of other StringBuffer methods
        return lastIndexOf(str, count);
    }

    /**
     * @since      1.4
     */
    @Override
    public synchronized int lastIndexOf(String str, int fromIndex) {
        return super.lastIndexOf(str, fromIndex);
    }

    /**
     * @since   JDK1.0.2
     */
    @Override
    public synchronized StringBuffer reverse() {
        toStringCache = null;
        super.reverse();
        return this;
    }

    @Override
    public synchronized String toString() {
        if (toStringCache == null) {
            toStringCache = Arrays.copyOfRange(value, 0, count);
        }
        return new String(toStringCache, true);
    }

    /**
     * Serializable fields for StringBuffer.
     *
     * @serialField value  char[]
     *              The backing character array of this StringBuffer.
     * @serialField count int
     *              The number of characters in this StringBuffer.
     * @serialField shared  boolean
     *              A flag indicating whether the backing array is shared.
     *              The value is ignored upon deserialization.
     */
    private static final java.io.ObjectStreamField[] serialPersistentFields =
    {
        new java.io.ObjectStreamField("value", char[].class),
        new java.io.ObjectStreamField("count", Integer.TYPE),
        new java.io.ObjectStreamField("shared", Boolean.TYPE),
    };

    /**
     * readObject is called to restore the state of the StringBuffer from
     * a stream.
     */
    private synchronized void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        java.io.ObjectOutputStream.PutField fields = s.putFields();
        fields.put("value", value);
        fields.put("count", count);
        fields.put("shared", false);
        s.writeFields();
    }

    /**
     * readObject is called to restore the state of the StringBuffer from
     * a stream.
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        java.io.ObjectInputStream.GetField fields = s.readFields();
        value = (char[])fields.get("value", null);
        count = fields.get("count", 0);
    }
}

#案例：
	package com.mk.coffee.test.jdk.resource;
	
	/**
	 * @Auther: makui
	 * @Date: 2018/12/17
	 * @Description: String源码分析
	 */
	public class String_Demo {
	    public static String abc ="abc";
	    public static String ABC = "ABC";
		public static String d;
	    public static StringBuffer sb = new StringBuffer("abc");
	    public static StringBuilder sbr = new StringBuilder("abc");
	
	    public static void main(String[] args) {
	        System.out.println(abc.compareTo(ABC));//32
	        System.out.println(String.CASE_INSENSITIVE_ORDER.compare(abc,ABC)); //不区分大小写判断
	        System.out.println(abc.hashCode()==ABC.hashCode()); //false
	        System.out.println(new String(sb).equals(new String(sbr)) ); //true String构造器可以由 StringBuffer StringBuilder构造
			System.out.println(sb.substring(1,3));//bc 索引位置从第2个开始，到第3个 不包含尾部
        	System.out.println(sb.substring(0,3));//abc 索引位置从第1个开始，到第3个
       	 	System.out.println(sb.substring(1));//bc
       	 	System.out.println(sb.substring(1,4));//java.lang.StringIndexOutOfBoundsException
			System.out.println(d.isEmpty()); //java.lang.NullPointerException 此处说明，String判断是否为空不能用String.isEmpty()去判断 应该用commons.lang3下的包StringUtils.isEmpty()去判断
	}
