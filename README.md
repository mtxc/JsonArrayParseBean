# JsonArrayParseBean
##结合jackson框架，实现n层实体类数组的递归解析
``` java
public static Object jsonParseResponse(JSONObject json, Class<?>... responses) {
		Object obj = null;
		if(responses.length > 1){
			try {
				obj = responses[0].newInstance();
				for(Field field : responses[0].getDeclaredFields()){
					if(field.getType().isAssignableFrom(ArrayList.class)){
						// 如果是数组, 递归解析
						JSONArray array = json.getJSONArray(field.getName());
						Object[] objects = (Object[]) Array.newInstance(responses[1], array.length());
						for(int i=0; i<objects.length; i++){
							objects[i] = jsonParseResponse(array.getJSONObject(i), Arrays.copyOfRange(responses, 1, responses.length));
						}
						List<?> list = asList(objects);
						Method m = responses[0].getMethod(ReflectUtil.setterNameFromField(field), list.getClass());
						m.invoke(obj, list);
					}else{
						// 如果不是数组，直接调用set方法
						Method m = responses[0].getMethod(ReflectUtil.setterNameFromField(field), field.getType());
						m.invoke(obj, json.get(field.getName()));
					}
				}
			} catch (InstantiationException e) {
				e.printStackTrace();
			} catch (IllegalAccessException e) {
				e.printStackTrace();
			} catch (JSONException e) {
				e.printStackTrace();
			} catch (NoSuchMethodException e) {
				e.printStackTrace();
			} catch (IllegalArgumentException e) {
				e.printStackTrace();
			} catch (InvocationTargetException e) {
				e.printStackTrace();
			}
		}else{
			try {
				obj = objectMapper.readValue(json.toString(), responses[0]);
			} catch (JsonParseException e) {
				e.printStackTrace();
			} catch (JsonMappingException e) {
				e.printStackTrace();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return obj;
	}
```
