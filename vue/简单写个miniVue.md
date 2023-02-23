# 简单写个 Vue

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

# 自己编写目标模板 mini-vue

## 方法 1

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Template</title>
  </head>
  <style>
    .bb {
      font-size: 20px;
    }
    .cc {
      color: blueviolet;
      cursor: pointer;
    }
  </style>
  <body>
    <div id="app"></div>
    <script type="text/javascript">
      let str = `<ul class="bb cc" @click="uu"><li @click="aaa">test:{{a}}</li><li>2</li><li>3<p>bbb</p></li></ul><div @click="uu">{{a}}<div>{{a}}</div><div><input type="text" v-model="a"></input>`;
      var originData = {
        a: 111,
        b: 222,
      };
      var dep = [];
      var oldVDom = {};
      var data = new Proxy(originData, {
        get(data, key, proxy) {
          console.log("get方法");
          return data[key];
        },
        set(data, key, newValue, proxy) {
          console.log("set方法");
          data[key] = newValue;
          render();
          return true;
        },
      });
      var count = 0;
      var methods = {
        uu() {
          // alert("uu");
          data.a = count;
          data.b = count;
          count++;
        },
        aaa() {
          // alert("aaa");
        },
      };
      render(true);

      function render(isFirst = false) {
        console.log("isFirst", isFirst);
        if (isFirst) {
          let vdom = generateTree(str);
          let t = generateHtml(vdom);
          console.log("render:", data);
          let app = document.querySelector("#app");
          app.innerHTML = ``;
          app.appendChild(t);
          oldVDom = vdom;
        } else {
          dep.forEach((item) => {
            item.update();
          });
        }
      }

      function generateHtml(root) {
        let r = createElement(root);

        if (root.children) {
          for (const node of root.children) {
            let child = generateHtml(node);
            r.appendChild(child);
          }
        }
        return r;
      }

      function createElement(node) {
        let r = {};
        if (node.tagName == "text") {
          r = document.createTextNode(node.text);
          let text = node.text.replace(/\{\{(\w+)\}\}/g, (findStr, $1) => {
            dep.push({
              update: () => {
                r.textContent = data[$1];
              },
            });
            return data[$1];
          });
          r.textContent = text;
        } else {
          r = document.createElement(node.tagName);
        }
        let props = node.props;
        if (props) {
          for (const prop of props) {
            if (prop.charAt(0) != "@" && prop.substring(0, 2) != "v-") {
              console.log("prop", prop);
              if (prop) {
                let propName = prop.split("=")[0];
                let classList = prop
                  .split("=")[1]
                  .replaceAll('"', "")
                  .split(" ");
                for (const className of classList) {
                  console.log("className: ", className);
                  r.classList.add(className);
                }
              }
            } else {
              console.log("front name", prop.split("=")[0]);

              let name = prop.split("=")[0];
              if (name == "v-model") {
                let value = prop.split("=")[1];
                // console.log("v-model", name, value);
                r.addEventListener("input", function () {
                  console.log("before", data, value.replaceAll('"', ""));

                  data[value.replaceAll('"', "")] = r.value;

                  console.log("change", data);
                });
              } else {
                let eventName = prop.split("=")[0].slice(1);
                let value = prop.split("=")[1];
                value = value.replaceAll('"', "");
                r.addEventListener(eventName, methods[value]);
              }
            }
          }
        }
        return r;
      }

      function generateTree(str) {
        let mark = "";
        let n = 0;
        let stack = [];
        let vm = [];
        let root = {
          tagName: "div",
          props: [],
          children: [],
          text: "",
        };
        for (let i = 0; i < str.length; i++) {
          let c = str.charAt(i);
          let next = str.charAt(i + 1);
          if (c == "<") {
            if (next == "/") {
              n = i;
              mark = "after";
            } else {
              n = i;
              mark = "before";
            }
          }
          if (c == ">") {
            if (mark == "before") {
              stack.push([n, i]);
            } else {
              let position = stack.pop();
              let obj = {};
              let classProp = str
                .substring(position[0] + 1, position[1])
                .match(/class=\"[\w\d\s\-]*\"/g);
              let props = str.substring(position[0] + 1, position[1]);
              if (classProp) {
                props = props.replaceAll(classProp, "");
              }
              props = props.split(" ").slice(1);
              if (classProp) {
                props.push(classProp[0]);
              }
              let tagName = str
                .substring(position[0] + 1, position[1])
                .split(" ")[0];
              let text = str.substring(position[1] + 1, n);
              let deep = stack.length;
              obj.tagName = tagName;
              obj.props = props;
              obj.children = [];
              obj.text = text;
              obj.deep = deep;
              vm.push(obj);
            }
          }
        }
        for (let i = 0; i < vm.length; i++) {
          let father = -1;
          for (let j = i + 1; j < vm.length; j++) {
            if (vm[i].deep - 1 == vm[j].deep) {
              father = j;
              break;
            }
          }
          if (father != -1) {
            vm[father].children.push(vm[i]);
          } else {
            root.children.push(vm[i]);
          }
        }
        dfs(root);
        return root;
      }

      function childrenSort(str) {
        let mark = "";
        let stack = [];
        let n = 0;
        let arr = [];
        for (let i = 0; i < str.length; i++) {
          let c = str.charAt(i);
          let next = str.charAt(i + 1);
          if (c == "<") {
            if (next == "/") {
              n = i;
              mark = "after";
            } else {
              n = i;
              mark = "before";
            }
          }
          if (c == ">") {
            if (mark == "before") {
              stack.push([n, i]);
            } else {
              let position = stack.pop();
              if (stack.length == 0) {
                arr.push([...position, n, i]);
              }
            }
          }
        }
        let ans = str;
        for (let i = 0; i < arr.length; i++) {
          const element = arr[i];
          ans = ans.replace(
            str.substring(element[0], element[3] + 1),
            "|" + "array:" + "[" + [...element, i] + "]" + "|"
          );
        }
        return ans.split("|");
      }

      function dfs(root) {
        for (let i = 0; i < root.children.length; i++) {
          const element = root.children[i];
          let str = element.text;
          let arr = childrenSort(str);
          console.log("arr:", arr);
          let children = [];
          for (let i = 0; i < arr.length; i++) {
            if (arr[i].includes("array:")) {
              let array = JSON.parse(arr[i].split("array:")[1]);
              children.push(element.children[array[4]]);
            } else {
              if (arr[i] != "") {
                console.log(
                  "text",
                  arr[i].replace(/\{\{(\w+)\}\}/g, function (findStr, $1) {
                    return data[$1];
                  })
                );
                // children.push({
                //   tagName: "text",
                //   text: arr[i].replace(
                //     /\{\{(\w+)\}\}/g,
                //     function (findStr, $1) {
                //       return data[$1];
                //     }
                //   ),
                // });
                children.push({
                  tagName: "text",
                  text: arr[i],
                });
              }
            }
          }
          dfs(element);
          element.children = children;
        }
      }
    </script>
  </body>
</html>
```

## 方法 2

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Template</title>
  </head>
  <style>
    .bb {
      font-size: 20px;
    }
    .cc {
      color: blueviolet;
    }
  </style>
  <body>
    <script type="text/javascript">
      let str = `<ul class="bb cc" @click="uu"><li @click="aaa">1</li><li>2</li><li>3<p>bbb</p></li></ul>`;
      var methods = {
        uu() {
          alert("uu");
        },
        aaa() {
          alert("aaa");
        },
      };
      let vdom = generateTree(str);
      console.log("vdom", vdom);
      let t = generateHtml(vdom);
      console.log("t", t);
      document.body.appendChild(t);

      function generateHtml(root) {
        let r = createElement(root);

        if (root.children) {
          for (const node of root.children) {
            let child = generateHtml(node);
            r.appendChild(child);
          }
        }
        return r;
      }

      function createElement(node) {
        let r =
          node.tagName == "text"
            ? document.createTextNode(node.text)
            : document.createElement(node.tagName);
        let props = node.props;
        if (props) {
          for (const prop of props) {
            if (prop.charAt(0) != "@") {
              console.log("prop", prop);
              if (prop) {
                let propName = prop.split("=")[0];
                let classList = prop
                  .split("=")[1]
                  .replaceAll('"', "")
                  .split(" ");
                for (const className of classList) {
                  console.log("className: ", className);
                  r.classList.add(className);
                }
              }
            } else {
              let eventName = prop.split("=")[0].slice(1);
              let value = prop.split("=")[1];
              // console.log(eventName,typeof value, methods[value], methods["aaa"]);
              console.log(eventName, value);
              value = value.replaceAll('"', "");
              r.addEventListener(eventName, methods[value]);
            }
          }
        }
        return r;
      }

      function generateTree(str) {
        let mark = "";
        let n = 0;
        let stack = [];
        let vm = [];
        let root = {
          tagName: "div",
          props: [],
          children: [],
          text: "",
        };
        for (let i = 0; i < str.length; i++) {
          let c = str.charAt(i);
          let next = str.charAt(i + 1);
          if (c == "<") {
            if (next == "/") {
              n = i;
              mark = "after";
            } else {
              n = i;
              mark = "before";
            }
          }
          if (c == ">") {
            if (mark == "before") {
              stack.push([n, i]);
            } else {
              let position = stack.pop();
              let obj = {};
              let classProp = str
                .substring(position[0] + 1, position[1])
                .match(/class=\"[\w\d\s\-]*\"/g);

              // let props = str
              //   .substring(position[0] + 1, position[1])
              //   .replaceAll(classProp, "")
              //   .split(" ")
              //   .slice(1);
              let props = str.substring(position[0] + 1, position[1]);
              if (classProp) {
                props = props.replaceAll(classProp, "");
              }
              props = props.split(" ").slice(1);
              if (classProp) {
                props.push(classProp[0]);
              }
              console.log("props: ", props);
              let tagName = str
                .substring(position[0] + 1, position[1])
                .split(" ")[0];
              let text = str.substring(position[1] + 1, n);
              let deep = stack.length;
              obj.tagName = tagName;
              obj.props = props;
              obj.children = [];
              obj.text = text;
              obj.deep = deep;
              vm.push(obj);
            }
          }
        }
        for (let i = 0; i < vm.length; i++) {
          let father = -1;
          for (let j = i + 1; j < vm.length; j++) {
            if (vm[i].deep - 1 == vm[j].deep) {
              father = j;
              break;
            }
          }
          if (father != -1) {
            vm[father].children.push(vm[i]);
          } else {
            root.children.push(vm[i]);
          }
        }
        dfs(root);
        // console.log(root);
        return root;
      }

      function childrenSort(str) {
        let mark = "";
        let stack = [];
        let n = 0;
        let arr = [];
        for (let i = 0; i < str.length; i++) {
          let c = str.charAt(i);
          let next = str.charAt(i + 1);
          if (c == "<") {
            if (next == "/") {
              n = i;
              mark = "after";
            } else {
              n = i;
              mark = "before";
            }
          }
          if (c == ">") {
            if (mark == "before") {
              stack.push([n, i]);
            } else {
              let position = stack.pop();
              if (stack.length == 0) {
                arr.push([...position, n, i]);
              }
            }
          }
        }
        let ans = str;
        for (let i = 0; i < arr.length; i++) {
          const element = arr[i];
          ans = ans.replace(
            str.substring(element[0], element[3] + 1),
            "|" + "array:" + "[" + [...element, i] + "]" + "|"
          );
        }
        return ans.split("|");
      }

      function dfs(root) {
        for (let i = 0; i < root.children.length; i++) {
          const element = root.children[i];
          let str = element.text;
          let arr = childrenSort(str);
          console.log("arr:", arr);
          let children = [];
          for (let i = 0; i < arr.length; i++) {
            if (arr[i].includes("array:")) {
              let array = JSON.parse(arr[i].split("array:")[1]);
              children.push(element.children[array[4]]);
            } else {
              if (arr[i] != "") {
                children.push({
                  tagName: "text",
                  text: arr[i],
                });
              }
            }
          }
          dfs(element);
          element.children = children;
        }
      }
    </script>
  </body>
</html>
```

## 方法 3

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
    <div id="app"></div>
  </body>
  <script>
    class vue {
      constructor(options) {
        this.el = document.querySelector(options.el);
        let _this = this;
        this.data = new Proxy(options.data, {
          get(data, key, proxy) {
            console.log("get", arguments);
            return data[key];
          },
          set(data, key, newValue, proxy) {
            console.log("set", arguments);
            data[key] = newValue;
            _this.render(options.template, key);
            options.updated();
            return true;
          },
        });
        this.render(options.template, "null");
        options.mounted();
      }
      render(template, key) {
        if (key == "null") {
          this.el.innerHTML = this.parse(template);
        } else {
          let nodeList = document.querySelectorAll(
            "." + key + "-data" + "-read"
          );
          for (let i = 0; i < nodeList.length; i++) {
            const element = nodeList[i];
            element.innerText = this.data[key];
          }
        }
      }
      parse(template) {
        let reg = /\{\{(.*)\}\}/;
        let key = template.match(reg)[1];
        template = template.replace(reg, this.data[key]);
        setTimeout(() => {
          let input = document.querySelector("input");
          let modelKey = template.match(/v-model\=\"(.*)\"/)[1];
          input.addEventListener("input", () => {
            this.data[modelKey] = input.value;
          });
        }, 0);

        return template;
      }
    }
    const app = new vue({
      el: "#app",
      template:
        '<div class="a-data-read">{{a}}</div><input v-model="a"></input>',
      data: {
        a: 123,
      },
      mounted() {
        // alert("mounted");
      },
      updated() {
        // alert("updated");
      },
    });
  </script>
</html>
```

# 模板引擎实现+建议内置指令

- 目前指令只编写了 v-model @click
- diff 算法未编写

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
    <div id="app"></div>
  </body>
  <script>
    class vue {
      constructor(options) {
        console.log("options", options);
        this.dep = [];
        this.options = options;
        this.method = options.method;
        this.data = this.reactive(options.data);
        this.options.created();
        const vdom = this.parse(options.template);
        console.log("vdom", vdom);
        this.render(vdom);
        this.options.mounted();
      }
      reactive(originData) {
        const _this = this;
        const proxyData = new Proxy(originData, {
          get(data, key, proxy) {
            console.log("get方法", data, key);
            return data[key];
          },
          set(data, key, newValue, proxy) {
            console.log("set方法", data, key, newValue);
            data[key] = newValue;
            _this.dep.forEach((item) => {
              item.update();
              _this.options.updated();
            });
            return true;
          },
        });
        return proxyData;
      }

      parse(html) {
        const stack = [];
        let root = {};
        for (let i = 0, index = 0; i < html.length; i++) {
          let char = html[i];
          if (char == "<") {
            let tag = "";
            index = i;
            while (char != ">") {
              tag += char;
              ++i;
              char = html[i];
            }
            let position = [index, i];
            tag += char;

            if (tag.includes("/")) {
              let node = stack.pop();
              node.closeTagPosition = position;
              node.tagName = this.getTagName(node);
              node.directives = this.getDirectives(node);
              node.className = this.getClassName(node);
              node.text = this.getText(node);
              if (stack.length == 0) {
                root = node;
              }
            } else {
              let node = {
                tagName: "",
                children: [],
                props: [],
                openTagPosition: position,
              };
              if (stack.length > 0) {
                let parent = stack[stack.length - 1];
                parent.children.push(node);
              }
              stack.push(node);
            }
          }
        }
        return root;
      }

      getTagName(node) {
        const template = this.options.template;
        const tagStr = template.substring(
          node.openTagPosition[0],
          node.openTagPosition[1] + 1
        );
        const tag = tagStr.split(" ")[0].substring(1).replace(">", "");
        return tag;
      }

      getDirectives(node) {
        const directives = [];
        const template = this.options.template;
        let tagStr = template.substring(
          node.openTagPosition[0],
          node.openTagPosition[1] + 1
        );
        console.log(tagStr);
        tagStr = tagStr.replace(">", "").replace("<", "");
        const arr = tagStr.split(" ");
        for (let i = 0; i < arr.length; i++) {
          const element = arr[i];
          if (element.includes("v-")) {
            let directiveArr = element.split("=");
            let directive = {
              name: directiveArr[0],
              value: directiveArr[1],
            };
            directives.push(directive);
          } else if (element.includes("@")) {
            let directiveArr = element.split("=");
            let directive = {
              name: directiveArr[0],
              value: directiveArr[1].replaceAll('"', ""),
            };
            console.log("element", directive);
            directives.push(directive);
          }
        }
        return directives;
      }

      getClassName(node) {
        let name = "";
        const template = this.options.template;
        const tagStr = template.substring(
          node.openTagPosition[0],
          node.openTagPosition[1] + 1
        );
        let classList = tagStr.match(/class=\"[\w\d\s\-]*\"/g);
        if (classList) {
          classList = classList[0].split("=")[1];
          classList = classList.replaceAll('"', "").split(" ");
          console.log("classList", classList);
        }
        return classList ? classList : [];
      }

      getText(node) {
        const template = this.options.template;
        let text = template.substring(
          node.openTagPosition[1] + 1,
          node.closeTagPosition[0]
        );
        const textObj = { text, after: -1, tagName: "text" };
        for (let i = 0; i < node.children.length; i++) {
          const child = node.children[i];
          const childNodeStr = this.options.template.substring(
            child.openTagPosition[0],
            child.closeTagPosition[1] + 1
          );
          if (text.indexOf(childNodeStr) <= 0) {
            textObj.after = i;
          }
          text = text.replace(childNodeStr, "");
          textObj.text = text;
        }
        if (textObj.text.length > 0) {
          node.children.splice(textObj.after + 1, 0, textObj);
        }
      }

      render(vdom) {
        const dom = this.getDom(vdom);
        this.options.el.appendChild(dom);
      }

      getDom(root) {
        const node =
          root.tagName == "text"
            ? document.createTextNode(root.text)
            : document.createElement(root.tagName);
        if (root.tagName != "text" && root.className.length > 0) {
          for (const classes of root.className) {
            node.classList.add(classes);
          }
        }
        if (root.tagName == "text") {
          const data = root.text.replace(
            /\{\{\s*(\w+)\s*\}\}/g,
            (findStr, $1) => {
              this.dep.push({
                update: () => {
                  console.log("调用", this.data[$1]);
                  node.textContent = this.data[$1];
                },
              });
              return this.data[$1];
            }
          );
          node.textContent = data;
        }
        this.eventMount(root, node);
        for (let i = 0; i < root.children?.length; i++) {
          const child = root.children[i];
          node.appendChild(this.getDom(child));
        }
        return node;
      }

      eventMount(node, dom) {
        for (let i = 0; i < node.directives?.length; i++) {
          const directive = node.directives[i];
          const key = directive.value.replaceAll('"', "").replace(">", "");
          if (directive.name == "v-model") {
            dom.addEventListener("input", () => {
              this.data[key] = arguments[1].value;
            });
          } else if (directive.name == "@click") {
            console.log("tnnd:", this.method[directive.value], directive.value);
            dom.addEventListener("click", this.method[directive.value]);
          }
        }
      }
    }

    const app = new vue({
      el: document.querySelector("#app"),
      template: `<div class="container"><input class="child-1" type="text" v-model="aa"></input>111<div class="child-2-1 child-2-2">{{ aa }}</div><p @click="cc">bbb</p></div>`,
      data: {
        aa: 1,
      },
      created() {
        console.log("created");
      },
      mounted() {
        console.log("mounted");
      },
      updated() {
        console.log("update");
      },
      method: {
        cc() {
          console.log("cc");
        },
      },
    });
  </script>
</html>


```
