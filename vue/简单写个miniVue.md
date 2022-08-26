# 简单写个Vue
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="app">
      {{name}}
      <div>
        {{age}}
        <div @click="clickTap">{{ooo.c.d.aaa}}</div>
        <div class="a" v-on:click="test">{{name}}</div>
      </div>
      <div>
        <input type="text" v-model="name" />
      </div>
    </div>
    <script>
      class watcher {
        constructor(el, Vnode, attr, vm) {
          this.el = el;
          this.Vnode = Vnode;
          this.attr = attr;
          this.vm = vm;
        }
        updated() {
          switch (this.attr) {
            case "text":
              this.el.textContent = this.vm.htmlOrData(
                this.Vnode,
                null,
                this.vm
              );
              break;
            case "input":
              this.el.value = this.vm.htmlOrData(this.Vnode, null, this.vm);
              break;
          }
        }
      }
      class Event {
        constructor(el, vm) {
          this.parent = el;
          this.compEvent(el);
        }
        compEvent(el) {
          [...el.attributes].forEach((v) => {
            let name = v.nodeName;
            if (name.includes("@") || name.includes("v-on")) {
              let e = name.includes("@")
                ? name.split("@")[1]
                : name.split(":")[1];
              this.on(el, e, v.nodeValue);
              el.removeAttribute(name.includes("@") ? `@${e}` : `v-on:${e}`);
            }
          });
        }
        on(el, event, callback) {
          el.addEventListener(event, this.parent[callback]);
        }
      }
      class Vue {
        constructor(options) {
          let data = options?.data ?? {};
          this.dep = [];
          for (let key in data) {
            this[key] = data[key];
          }
          this.initState(this);
          this.initMethod(options);
          this.hook(options?.el, options);
        }
        nodeList(el) {
          let dom = el.childNodes;
          let Vnode = this;
          [...dom].forEach((v) => {
            let reg = /\{\{(.*)\}\}/;
            let text = v.textContent;
            if (v.nodeType == 1 && v.tagName != "INPUT") {
              this.nodeList(v);
              new Event(v, this);
            } else if (v.nodeType == 3 && reg.test(text)) {
              v.textContent = this.htmlOrData(RegExp.$1, v, Vnode);
              this.dep.push(new watcher(v, RegExp.$1, "text", this));
            } else if (v.nodeType == 1 && v.type == "text") {
              this.notice(v, v.attributes);
            }
          });
        }
        notice(v, attr) {
          Array.from(attr).forEach((o) => {
            if (o.nodeName == "v-model") {
              v.value = this.htmlOrData(o.nodeValue, v, this);
              this.dep.push(new watcher(v, o.nodeValue, "input", this));
              v.addEventListener("input", (e) => {
                this.setValue(o.nodeValue, e.srcElement.value);
              });
            }
          });
        }
        setValue(valStr, value) {
          let valArr = valStr.split(".");
          valArr.reduce((data, item, index, arr) => {
            if (index === arr.length - 1) {
              data[item] = value;
            }
            return data[item];
          }, this);
        }
        htmlOrData(v, dom, data) {
          let value = v.split(".");
          return value.reduce((total, next) => {
            return total[next];
          }, data);
        }
        hook(el, op) {
          if (op.created) {
            op.created.call(this);
          }
          if (el) {
            this.el = document.querySelector(el);
            this.nodeList(this.el);
            if (op.mounted) {
              op.mounted.call(this);
            }
          }
        }
        initState(data, vue) {
          if (!data || typeof data !== "object") {
            return;
          }
          Object.keys(data).forEach((key) => {
            this.obServer(data, key, data[key]);
          });
        }
        initMethod(options) {
          if (options.methods) {
            for (let e in options.methods) {
              this[e] = options.methods[e].bind(this);
            }
          }
        }
        obServer(data, key, value) {
          this.initState(value);
          let _this = this;
          Object.defineProperty(data, key, {
            get: function () {
              return value;
            },
            set: function (newValue) {
              value = newValue;
              _this.dep.forEach((v) => v.updated());
            },
          });
        }
      }
    </script>
    <script>
      var V = new Vue({
        el: "#app",
        data: {
          name: "shuhan",
          ooo: {
            a: 1,
            c: {
              d: {
                aaa: "aaa111",
              },
            },
          },
          list: [{ value: 1 }],
          age: "age222",
        },
        methods: {
          clickTap(e) {
            console.log(this.e);
          },
          test() {
            console.log(222);
          },
        },
        created() {
          this.test();
          console.log(this);
        },
        mounted() {},
      });
    </script>
  </body>
</html>

```