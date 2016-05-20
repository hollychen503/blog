# vuejs + i18next 实现多国语言UI

## 0. install i18next & i18next-xhr-backend
npm install i18next --save-dev
npm install i18next-xhr-backend --save-dev


## 1.在main.js 导入 i18next 作为全局包，避免在每个component 单独import，浪费资源
main.js

```
import i18next from 'i18next';
import XHR from 'i18next-xhr-backend';
i18next
.use(XHR)
.init({
  backend:{     loadPath: '/locales/{{lng}}/translation.json'   },
  lng: 'en',
  fallbackLng:'en',  
}, (err, t) => {
  // initialized and ready to go!
});
```

将 i18next  放入全局对象 router
```
router.funconfig = {};
router.funconfig.i18next = i18next;
```

## 2. 在顶层 vue component 中定义一个变量作为语言切换触发器。
当变换语言时，修改这个变量（以便引起连锁反应）
例如，
### a. 在App.vue 里 定义一个变量i18nlang ：
```
  data () {
    return {
      i18nlang : "",
```      
### b. 将变量传递给子 组件：
```
      <sidebar 
        :i18nlang="i18nlang"
      >
      </sidebar>        
```

### c. 当切换语言时，修改变量i18nlang
```
var that=this;
i18next.changeLanguage('zh', (err, t) => {
    console.log('changeLanguage:' + err);
    that.i18nlang += " ";
    if (that.i18nlang.length>5) that.i18nlang="";
}); 
```

## 3. 在其他vue 组件中，用计算变量的方法产生多国语言文字
### a. 引入 外部变量
```
export default {
	props: ['i18nlang'],
```

### b. 用计算变量产生实际的语言文字。
```
computed:{
    tArrival: function() {
        var i18next= this.$route.router.funconfig.i18next;
        return (i18next.t('key', {defaultValue: 'Arrival'}) + this.i18nlang).trim();
        // 使用 defaultValue，防止英文语言文件读取失败的情况； 
    },
```    
在template 中使用这个文字
```
<tr><td colspan="2">{{ tArrival }}/</td></tr>
```

## 注意：
1. 如果要用backend 读取外部翻译，就不能在i18next.init 中初始化 resources 的东西，否则backend不工作！
