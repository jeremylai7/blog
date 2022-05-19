# MD5加密算法

## 第一种：使用jdk自带的java.security.MessageDigest实现
```
String plainText = "admin";
        byte[] secretBytes = null;
        try {
            secretBytes = MessageDigest.getInstance("md5").digest(
                    plainText.getBytes());
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("没有这个md5算法！");
        }
        String md5code = new BigInteger(1, secretBytes).toString(16);
        for (int i = 0; i < 32 - md5code.length(); i++) {
            md5code = "0" + md5code;
        }
        System.out.println(md5code);
```
## 第二种：使用spring自带的DigestUtils实现
```
DigestUtils.md5DigestAsHex("admin".getBytes())
```
