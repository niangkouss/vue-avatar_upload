###avatar头像截取组件

####使用
在main.js里面引入
```javascript
import AvatarUpload from './plugins/avatar_upload'
Vue.use(AvatarUpload);

```
```html
使用结构:
<avatar-upload v-model="result"></avatar-upload>
```
result可以获得截取图片之后的base64码,之后上传服务器,服务器处理后返回显示头像
#### 显示上传图片的框默认边长是200px,可以修改,通过传入canvasShowLength
```html
<avatar-upload v-model="result" :canvasShowLength="500"></avatar-upload>
```
#### 显示截取后图片的框的边长,默认100px,可以修改,通过传入saveCanvasLength
```html
<avatar-upload v-model="result" :saveCanvasLength="200"></avatar-upload>
```
#### 限制图片的上传大小,默认是1MB,可以修改,通过传入limitFileSize,如2MB,传入2
```html
<avatar-upload v-model="result" :limitFileSize="1"></avatar-upload>
```
