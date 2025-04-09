---
title: ast-js模板
date: 2025-03-21 00:19:38
category: 爬虫
tags: AST
cover: img/spider.png
---
# ast_js_template
```Javascript
let parse = require('@babel/parser').parse;
let generate = require('@babel/generator').default;
let traverse = require('@babel/traverse').default;
let types = require('@babel/types');
let fs = require('fs');

let input = fs.readFileSync('input.js', 'utf8').toString();
let ob_ast = parse(input);
let ast = parse(input);

/**
 * ob混淆: 大数组, 数组位移, 解密函数, 业务代码
 */
// 0. 解密函数名
let $_ob = '';
// 1.1 提取ob混淆代码
traverse(ob_ast, {
    Program: function (path) {
        $_ob = path.get('body.2.declarations.0.id').node.name
        path.stop();
        // 将业务代码删除, 只保留ob混淆代码
        let len = path.get('body').length - 3;
        for (let i = 0; i < len; i++) {
            path.get('body')[3].remove();
        }
    },
});

// 1.2 注入内存
/**
 * function  generate
 * retainLines  尝试输出代码中使用与源代码（第三个参数）一样的行号  默认 false
 * comments 输出结果是否包含注释, 默认: true
 * minified 是否压缩输出, 默认: false
 */
eval(generate(ob_ast, { minified: true }).code);

// 1.4 去除ob混淆代码
traverse(ast, {
    Program: function (path) {
        path.stop();
        // 删除ob混淆代码
        path.get('body')[0].remove();
        path.get('body')[0].remove();
        path.get('body')[0].remove();
    },
});

// 1.5 利用解密函数将字符串还原成明文
traverse(ast, {
    // 调用表达式
    CallExpression: function (path) {
        // 调用函数名=_$ob
        if (path.node.callee.name === $_ob) {
            // 源调用表达式 替换成 执行调用返回的结果
            path.replaceInline(types.valueToNode(eval(path.toString())));
        }
    },
});

/**
 * 2. 花指令还原
 * a. 成员属性: ObjectProperty
 * var obj = {
 *     "member": function (c, d) {
 *          return c === d;
 *     },
 * }
 * b. 赋值表达式: AssignmentExpression
 * var obj = {};
 * obj["member"] = function (c, d) {
 *      return c === d;
 * }
 */
// 2.1 先进行字符串拼接
traverse(ast, {
    // 字符串拼接, 防止member被拆分
    BinaryExpression: {
        exit: function (path) {
            const { confident, value } = path.evaluate();
            if (confident) {
                path.replaceInline(types.valueToNode(value));
            }
        },
    },
});

// 需要还原的obj
const obj_list = [请修改];
traverse(ast, {
    // 调用表达式(obj['member'](args))
    CallExpression: {
        exit: function (path) {
            // 判断function_name是否为成员表达式
            if (
                path.get('callee').isMemberExpression() &&
                obj_list.includes(path.get('callee.object').node.name)
            ) {
                // 获取成员表达式主体对象, 属性值, 参数(obj['member'](args))
                let obj = path.get('callee.object').node.name;
                let member = path.get('callee.property').node.value;
                let args_node_arr = path.node.arguments;
                // 获取成员表达式对象的作用域, 再取其path进行遍历
                let obj_path = path.scope.getBinding(obj).path;
                if (obj_path.get('init').isIdentifier()) {
                    obj_path = path.scope.getBinding(obj).scope.path;
                }
                obj_path.traverse({
                    // 成员属性
                    ObjectProperty: function (path_) {
                        // 判断左右值
                        if (
                            path_.get('key').isStringLiteral() &&
                            path_.node.key.value === member &&
                            path_.get('value').isFunctionExpression()
                        ) {
                            // 获取返回值
                            let return_path = path_.get('value.body.body.0.argument');
                            if (return_path.isBinaryExpression()) {
                                // 返回值为二进制表达式
                                let operator = return_path.node.operator;
                                let left_node = args_node_arr[0];
                                let right_node = args_node_arr[1];
                                path.replaceInline(
                                    types.binaryExpression(operator, left_node, right_node)
                                );
                            } else if (return_path.isCallExpression()) {
                                // 返回值为调用表达式
                                let func_node = args_node_arr[0];
                                let args_node = args_node_arr.slice(1);
                                path.replaceInline(types.callExpression(func_node, args_node));
                            }
                        }
                    },
                    // 赋值表达式
                    AssignmentExpression: function (path_) {
                        // 判断左右值
                        if (
                            path_.get('left').isMemberExpression() &&
                            path_.node.left.property.value === member &&
                            path_.get('right').isFunctionExpression()
                        ) {
                            // 获取返回值
                            let return_path = path_.get('right.body.body.0.argument');
                            if (return_path.isBinaryExpression()) {
                                // 返回值为二进制表达式
                                let operator = return_path.node.operator;
                                let left_node = args_node_arr[0];
                                let right_node = args_node_arr[1];
                                path.replaceInline(
                                    types.binaryExpression(operator, left_node, right_node)
                                );
                            } else if (return_path.isCallExpression()) {
                                // 返回值为调用表达式
                                let func_node = args_node_arr[0];
                                let args_node = args_node_arr.slice(1);
                                path.replaceInline(types.callExpression(func_node, args_node));
                            }
                        }
                    },
                });
            }
        },
    },
});
// 2.3 花指令还原-字符串
traverse(ast, {
    // 成员表达式(obj['member'])
    MemberExpression: {
        exit: function (path) {
            let obj = path.node.object.name;
            if (obj_list.includes(obj) && path.get('property').isStringLiteral()) {
                let member = path.get('property').node.value;
                // 获取成员表达式的作用域, 再取其path进行遍历
                let obj_path = path.scope.getBinding(obj).path;
                if (obj_path.get('init').isIdentifier()) {
                    obj_path = path.scope.getBinding(obj).scope.path;
                }
                obj_path.traverse({
                    // 成员属性
                    ObjectProperty: function (path_) {
                        // 判断左右值
                        if (
                            path_.get('key').isStringLiteral() &&
                            path_.node.key.value === member &&
                            path_.get('value').isStringLiteral()
                        ) {
                            // 替换成: 目标字符型字面量
                            path.replaceInline(types.valueToNode(path_.node.value.value));
                        }
                    },
                    // 赋值表达式
                    AssignmentExpression: function (path_) {
                        // 判断左右值
                        if (
                            path_.get('left').isMemberExpression() &&
                            path_.node.left.property.value === member &&
                            path_.get('right').isStringLiteral()
                        ) {
                            // 替换成: 目标字符型字面量
                            path.replaceInline(types.valueToNode(path_.node.right.value));
                        }
                    },
                });
            }
        },
    },
});

// 3. 代码优化
let beautiful = parse(generate(ast).code);

traverse(beautiful, {
    // 清除无引用代码
    VariableDeclarator: {
        enter: function (path) {
            const binding = path.scope.getBinding(path.node.id.name);

            if (
                binding &&
                binding.constantViolations.length === 0 &&
                !binding.referenced
            ) {
                // 变量存在 && 变量未被修改 && 变量未被引用
                path.parentPath.remove();
            }
        },
    },
});
traverse(beautiful, {
    // 字符串还原
    StringLiteral: {
        exit: function (path) {
            !!path.node.extra &&
            (path.node.extra.raw = JSON.stringify(path.node.value));
        },
    },
    // 进制转换
    NumericLiteral: {
        exit: function (path) {
            !!path.node.extra &&
            (path.node.extra.raw = JSON.stringify(path.node.value));
        }
    }
});
traverse(beautiful, {
    // 表达式还原
    ['BinaryExpression|CallExpression|ConditionalExpression']: {
        exit: function (path) {
            const { confident, value } = path.evaluate();
            if (confident) {
                path.replaceInline(types.valueToNode(value));
            }
        },
    },
});
traverse(beautiful, {
    // 删除冗余逻辑代码
    IfStatement: function (path) {
        if (path.get("test").isBooleanLiteral() || path.get("test").isNumericLiteral()) {
            if (path.node.test.value) {
                path.replaceInline(path.node.consequent.body);
            } else {
                if (path.node.alternate) {
                    path.replaceInline(path.node.alternate.body);
                } else {
                    path.remove()
                }
            }
        }
    },
});

let output = generate(beautiful).code;
fs.writeFileSync('output.js', output);

```
