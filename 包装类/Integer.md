# Integer
## toString(int i, int radix),将10进制数字转为指定进制的字符串，最大支持32进制
```java
    /**
     * All possible chars for representing a number as a String
     */
    final static char[] digits = {
            '0', '1', '2', '3', '4', '5',
            '6', '7', '8', '9', 'a', 'b',
            'c', 'd', 'e', 'f', 'g', 'h',
            'i', 'j', 'k', 'l', 'm', 'n',
            'o', 'p', 'q', 'r', 's', 't',
            'u', 'v', 'w', 'x', 'y', 'z'
    };
    
/**
     * Returns a string representation of the first argument in the
     * radix specified by the second argument.
     *
     * <p>If the radix is smaller than {@code Character.MIN_RADIX}
     * or larger than {@code Character.MAX_RADIX}, then the radix
     * {@code 10} is used instead.
     *
     * <p>If the first argument is negative, the first element of the
     * result is the ASCII minus character {@code '-'}
     * ({@code '\u005Cu002D'}). If the first argument is not
     * negative, no sign character appears in the result.
     *
     * <p>The remaining characters of the result represent the magnitude
     * of the first argument. If the magnitude is zero, it is
     * represented by a single zero character {@code '0'}
     * ({@code '\u005Cu0030'}); otherwise, the first character of
     * the representation of the magnitude will not be the zero
     * character.  The following ASCII characters are used as digits:
     *
     * <blockquote>
     * {@code 0123456789abcdefghijklmnopqrstuvwxyz}
     * </blockquote>
     * <p>
     * These are {@code '\u005Cu0030'} through
     * {@code '\u005Cu0039'} and {@code '\u005Cu0061'} through
     * {@code '\u005Cu007A'}. If {@code radix} is
     * <var>N</var>, then the first <var>N</var> of these characters
     * are used as radix-<var>N</var> digits in the order shown. Thus,
     * the digits for hexadecimal (radix 16) are
     * {@code 0123456789abcdef}. If uppercase letters are
     * desired, the {@link java.lang.String#toUpperCase()} method may
     * be called on the result:
     *
     * <blockquote>
     * {@code Integer.toString(n, 16).toUpperCase()}
     * </blockquote>
     *
     * @param i     an integer to be converted to a string.
     * @param radix the radix to use in the string representation.
     * @return a string representation of the argument in the specified radix.
     * @see java.lang.Character#MAX_RADIX
     * @see java.lang.Character#MIN_RADIX
     */
    public static String toString(int i, int radix) {
        //如果进制大于或者小于支持的进制范围则默认为10进制
        //Character.MIN_RADIX=2
        // Character.MAX_RADIX=36
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;

        /* Use the faster version */

        //如果是10进制，因为入参i本身就是10进制的，无需处理，直接返回数据
        if (radix == 10) {
            return toString(i);
        }

        //用于存放计算过程中的各个位数的字符串，从代码可以看出，总长度不超过33，出去正负数标志位，最大支持32位
        char buf[] = new char[33];
        //判断是否是负数
        boolean negative = (i < 0);
        //最大支持32位
        int charPos = 32;

        //如果不是负数则改为负数
        if (!negative) {
            i = -i;
        }

        //开始计算，如果值i小于进制数（不用进一位）则继续计算
        while (i <= -radix) {
            //将每一位的值存入buf[]对应的位中，最小位的值在数组最末尾
            //余数就是当前计算位的值
            buf[charPos--] = digits[-(i % radix)];
            //通过处于进制获取上一位的值
            i = i / radix;
        }
        //最后剩余的值就是最高位的值
        buf[charPos] = digits[-i];

        //如果是负数则在最高位的前一位加上富豪
        if (negative) {
            buf[--charPos] = '-';
        }
        //将buf中各个位的字符串拼接成字符串
        return new String(buf, charPos, (33 - charPos));
    }
```
