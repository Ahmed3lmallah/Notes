### [String Buffer](https://www.geeksforgeeks.org/stringbuffer-class-in-java/)

Using `StringBuffer` to reverse strings example:

	 public Map<String, Object> parseMap(Map<String, Object> map) {
        //look for a key
        StringBuffer stringBuffer = new StringBuffer(map.get("key").toString());
        //reverse and replace
        map.put("key",stringBuffer.reverse().toString());
        //return
        return map;
    }