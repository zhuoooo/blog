# 前端自动化测试实践

> 测试代码并非编码结束后锦上添花的一个孤立活动。它其实是一个系统工程，必须在设计阶段就做好充足准备，并通过后续一整套活动配合，最终产生成效。

<img src="./img/auto_test.jpg" />

## 前端需要写测试代码吗？

个人认为是需要的。



#### 为什么以前没有要求写测试代码？

结合我们团队的情况来看，主要有几个原因：

1. 成本很高，后期维护的成本也高；
2. 项目孵化阶段，以快为主（项目经理：别和我扯这些，你们可别瞎搞导致延期）；
3. 就写一个简单的页面，用不着写测试代码；
4. 团队人员稚嫩（没有这方面的意识），写代码都够呛，就别白日做梦了；
5. 前期测试人力充裕，能够通过手动执行大量测试用例来覆盖场景；
6. ...



#### 前端编写测试代码，到底需要测什么？

主要测试组件的功能和组件集成功能、以及核心场景流程，也就是我们常说的单元测试、集成测试、端到端测试。由于前端的代码无论如何拆分都是会和 DOM 发生关联，所以我们还需要加多一个**组件测试**。顺便一提，我们放弃了可视化层面（样色对不对，位置准不准）的测试，就是只测功能，举个例子：确认按钮，只需要测它能不能点击、触发对应的事件即可。



#### 测试用例从何而来？

测试用例不是开发个人 YY 出来的，所有的用例都是通过精心评审的。主要来源：

1. 首先，开发同事会在设计文档中的【场景分析】环节考虑所有场景；然后在文档评审的环节，我们会对用例的合理性进行评估，确认没有遗落；
2. 然后，开发和测试同事双方确认，这些用例能够覆盖全部场景；这一步测试同事会补充一部分用例；
3. 如果说最后还是有问题遗留到了转集成环节，那么修复 BUG 的时候，要将其转化成对应的测试用例；

以上基本能覆盖所有的核心场景。



#### 测试代码能带来什么？

客观上：

1. 手动测试的成本减少；
2. 机器执行比人执行更靠谱；
3. 测试的速度更快、覆盖范围更广：改一行代码可以有全量测试用例覆盖；
4. 用例可重复执行；
5. 功在当代，利在千秋；



主观上：

1. 给开发人员带来重构的底气：这点十分重要，关乎到整个项目代码的质量。好的代码是需要不断打磨的，想要提高项目代码的生命力，就需要不断进行重构；有了底气就能帮助开发养成优化代码的好习惯。
2. 项目架构演进提供保障：能覆盖全量测试用例，团队在架构演进上不需要再束手束脚。可以拍胸脯和项目经理说，这次的升级绝对没有问题。
3. 提升前端开发的能力：测试用例从设计文档而来，测试代码覆盖功能。这些都是要在不断的版本中去锻炼，只有锻炼才会有提升。



#### 测试代码是否会成为维护的负担？

会存在一定成本，但是我们推荐先修改测试代码成需求变更后的样子，让测试失败，再修改产品代码使其通过。这样你就不是在维护测试用例，而是在利用测试用例。



#### 所有前端有团队都要落实写测试吗？



#### 怎么说服项目经理？



## 基于 `Cypress` 的实践



### 分级测试策略

前端 UI 层，根据自身的特性分为单元测试与业务模块测试，持续集成。



#### 单元测试

测试范围：公共组件、hooks、函数

测试方式：基于 cypress 的 component 模式测试

执行策略：所有分支，集成在 gitlab CI / 千流

命名规则：`**/**.cy.ts`



#### 业务模块测试

测试范围：模块设计中的场景 / store 模式的 3、4 层用例，（对于一些操作结果，只断言到 `API` 调用参数是否正确）

测试方式：基于 cypress 的 e2e 模式测试

执行策略：根据用例中的标签，根据不同环境变量执行对应用例（如按照测试类型，bvt、level ...， 按照模块，集群、云主机 ...）

命名规则：`**/**.e2e.ts`



#### 如何按需执行用例

使用 `cypress-grep` 插件，在用例书写的时候加上 `tags`， 然后在 ci 中根据环境变量判断执行的用例

```ts
it('works as an array', { tags: ['@bvt', '@network'] }, () => {
  expect(true).to.be.true
})
```



### 基于 UI 组件的测试策略

UI 自动化测试，实际上所有的操作、断言都是基于页面元素与网络请求，而其中的页面元素在前端完全组件化的基础上，最终都是基于封装好了的基础组件。



#### 测试工具库

得益于 `cypress` 的 `command` 特性，对所有基础组件（`element-ui`、`idux`）的操作、断言进行二次封装，得到更贴近于开发使用的测试工具库 `@idux-command`。

这样在开发写测试用例时，就尽可能的不用关注最终页面元素形态以及组件内部封装细节，直接使用封装好了的 `command` 进行调用即可。

一个实际的例子

```ts
// demo.e2e.ts
// 选中表格中的一行，点击编辑按钮，判断打开出来的弹窗某个输入框提示是否正常
it('demo', () => {
    const TABLE = '[utid=page-table]';
    
    cy.vTable_getRow(TABLE, { text: 'demo-text' })
        .vButton_click('[utid=more]')
        .vButton_click('[utid=grid-menu] [utid=edit]')
        .vModal_isVisible('[utid=single-edit-vm-win]')
        .vMask_wait('[utid=single-edit-vm-win]')
        .vTab_selectTab('.single-edit-vm-win_tab', '配置')
        .vInput_containTip('[utid=disk-size]', '快速派生镜像生成的云主机不支持编辑磁盘大小');
});
```

开发不需要关注表格或者是标签组件的具体实现，只需要在编码的时候，对这些**具有交互型的组件**外层写好预先定义好的 `utid` 属性。那么在写用例时候就只是对这些**组件的操作**进行依次调用，更加贴合组件化的思想，而不用在页面上导出找元素定位。

保证工具库的 `api` 一致性，设计成 `v[组件名]_[方法](selector, [options])` 的形式。 如 `vModal_isVisible('[utid=win]')`

### 如何测试

#### 功能介绍

`cypress` 的部分特性

- **查询元素**，使用了 `jQuery` 来进行 `DOM` 元素的操作，所以有大量的如 `find`、`parent`、`next`  查询遍历方法可使用

  ```ts
  // 获取 ul 元素
  cy.get('ul')
  	// 获取 ul 下所有显示的 li 元素
  	.find('li:visible')
  ```

- **自动重试**，对于一些查询、断言指令，如 `get`，`should` 等，会根据设定好的时间不停的重试直到完成指令

  ```ts
  // 获取 ul 元素，如果没有显示，不超时的情况下会重试到显示为止
  cy.get('ul').should('be.visible')
  ```

- **网络拦截**，提供 `intercept` 指令对发起的请求进行拦截，可以对 `request`, `response` 修改、断言等操作

  ```ts
  // 拦截 /list 请求
  // 将返回值 mock 为 `{ data: [] }`
  // 添加别名 `getList`
  cy.intercept('GET', '/list', { data: [] }).as('getList');
  
  // 等待 `getList` 请求完毕
  cy.wait('@getList')
  	// 断言请求的 URL
      .its('request.url')
      .should('have.string', '/list')
  	// 断言返回的状态码为 200
      .its('response.statusCode')
      .should('eq', 200)
  ```

- **函数拦截**，提供 `spy`，`stub` 对函数的行为做拦截，用来断言函数是否执行，参数与返回值是什么，或者拦截函数的默认行为

  ```ts
  // 常见的一个触发事件场景
  cy.mount(PageTabs, {
      propsData: {
          value: 'mod2',
          tabs: [
              {
                  key: 'mod1
              },
              {
                  key: 'mod2',
              },
          ],
      },
      listeners: {
          // 创建一个别名为 inputSpy 的函数，赋值给 input
          input: cy.spy().as('inputSpy'),
      },
  });
  
  // 点击 mod1 
  cy.get('[data-href=mod1]').click();
  
  // inputSpy 应该触发，且参数为 mod1
  cy.get('@inputSpy').should('have.been.calledWith', 'mod1');
  ```

- **断言**，分为显式断言跟隐式断言

  - 隐式断言，指的是 `cypress` 的很多内置指令就已经包含了断言，如 `get`, `find` 这种，如果查找不到对应元素，那么就会失败

  - 显式断言，就是 `should` 这种语义化的断言语句，如

    ```ts
    // 判断 button 是否有 is-disabled 这个 class
    cy.get('button').should('have.class', 'is-disabled');
    cy.get('button').should('not.have.class', 'is-disabled');
    
    // 也可以传回调函数
    cy.get('button').should($el => {
        // 获取到 $el 元素，取自定义的属性值
        const tip = $el.attr('data-hint') || $el.attr('data-tip');
       	// 使用 expect 断言值是否相等
        expect(tip).equal('提示');
    });
    ```

- **异步队列**，所有 `command` 指令都是异步执行的，也就是说，在运行到 `cy.xxx` 的时候它是还没执行的，只会添加到异步队列内，然后等所有指令扫描完毕，才会依次执行

  ```ts
  // Bad
  it('test', () => {
      let username = undefined;
      cy.get('.user-name').then($el => {
          username = $el.text();
      });
  
      // 这里会优先上面的 then 执行，所以恒等于 undefined
      if (username) {
          cy.contains(username).click();
      } else {
          cy.contains('My Profile').click();
      }
  });
  
  // Good
  it('test', () => {
      let username = undefined;
      cy.get('.user-name').then($el => {
          username = $el.text();
          
          // 放在里面
          if (username) {
              cy.contains(username).click();
          } else {
              cy.contains('My Profile').click();
          }
      });
  });
  ```

  [Commands Are Asynchronous](https://docs.cypress.io/guides/core-concepts/introduction-to-cypress#Commands-Are-Asynchronous)  异步队列介绍

#### 最佳实践

更多例子：[Github Blog](https://github.com/bahmutov/cypress-examples)

##### 选择元素

组件元素使用 `utid` 属性标识选择，如

```ts
// <sf-button utid="create"></sf-button>

cy.get('[utid=create]').click();
```

在页面操作都是需要先命中对应的元素，使用与业务无关的 `utid` 属性，职责明确，能够拥有更好的维护性

##### 等待

尽量不使用 `cy.wait(number)` 这种形式，使用别名或者一些断言来替代，避免无效的等待时间

1. 等待某个接口加载完毕，使用 `as` 取一个别名，然后使用 `wait(@alias)` 等待完成

   ```ts
   cy.intercept('/tree').as('getTreeApi');
   
   cy.wait('@getTreeApi')
       .vTree_getItems(TREE_SELECTOR)
       .should('have.length', 33)
   ```

2. 等待某个元素显示后再进行操作，如等待页面主体元素 `.sf-layout-main` 渲染完成，这种时候就可以使用断言来处理

   `cypress` 会对断言进行不停的轮询重试，直到满足条件

   ```ts
   cy.get('.sf-layout-main').should('be.visible')
   ```

3. 一些定时器的场景，如轮询刷新，使用 `cy.clock` 加 `cy.tick` 去把时钟给 mock 掉

   ```ts
   cy.intercept('GET', '/list').as('getList');
   cy.clock()
       .wait('@getList')
   	// 触发下一次请求
       .tick(5000)
       .wait('@getList')
       .get('@getList.all')
       .should('have.length', 2)
   ```

   

4. 对于一些其他特殊需要 wait 的操作可使用 `cy.waitUtil` ，如表格查找元素需要每次滚动一段距离直到元素显示

   ```ts
   let num = 0;
   getTable(selector)
       .waitUntil($el => { # 只有返回为 true 的时候，才会接着往下执行
           if ($el!.find(rowSelector).length) {
               return $el;
           }
   
           // 开启了虚拟滚动时可能不在当前视图内，每次滚动一段距离
           num += 200;
           $el!.find('.sfis-grid-scroll-y').get(0).scroll(0, num);
   	})
   ```

   > wait-until 指令是通过 cypress-wait-until 插件支持的

##### 网络请求

使用 `cy.intercept` 拦截请求做一些 `mock` 、断言 等处理

1. 拦截请求返回值，做不同处理

   ```ts
   it('配置 api 返回空数据，显示为空', () => {
       cy.intercept('GET', '/tree', []).as('getTreeApi');
       cy.mount(UseTreeStore)
           .wait('@getTreeApi')
           .vTree_isEmpty(TREE_SELECTOR);
   });
   it('配置 api 返回正常数据，加载显示正确', () => {
       cy.intercept('GET', '/tree', treeMockData).as('getTreeApi');
       cy.mount(UseTreeStore)
           .wait('@getTreeApi')
           .vTree_getItems(TREE_SELECTOR)
           .should('have.length', 33)
   });
   ```

2. 表格查询场景，输入后触发搜索，断言查询参数中是否有 `query` 字段

   ```ts
   cy.intercept('GET', '/list*'}).as('getList');
   cy.get('[utid=search]')
       .type('xxx{enter}')
   	.wait('@getList')
   	.its('request.url')
       .should('have.string', 'query=xxx');
   ```

##### 时钟

对于一些使用 `setTimeout` 的场景，比如经常会遇到防抖、节流，会导致在操作后直接接断言，然后失败。

这种情况下可使用 `cy.clock` 与 `cy.tick` 去处理，如

```ts
// input 组件的校验场景

/** 输入文本 */
vInput_typeValue: (selector: string, value: string) => {
    // 组件库的 校验 有 setTimeout，使用 clock mock 处理
    cy.then(function () {
        if (!this.clock) {
            cy.clock();
        }
    });

    return getInputEl(selector).type(value).tick(300);
}
```

#### 如何做组件测试

[如何使用Cypress 进行TDD 组件测试 - 掘金](https://juejin.cn/post/7106672456911290382)

##### 组件测试测什么？

从使用这个组件的开发人员的角度看，测 `props`, `events` , `slot` 是否正确，测内部一些实现逻辑是否正常

从最终用户的角度看，测在页面上使用这个组件的操作行为是否符合预期，比如用户在一个步骤条中，点击上一步、下一步按钮分别应该得到什么结果

##### 组件单元测试

单元测试通常的三个阶段

安排（Arrange），执行（ Act），断言（Assert）

1. 初始化准备，一些基本参数
2. 执行某一些操作行为
3. 断言上述的行为是否符合预期

```ts
it('步骤条初始化与按钮操作正常', () => {
    
    // Arrange -> 初始化数据准备
    const options: StepOption[] = [
        {
            key: 'step1',
            label: 'step1-label',
            component: { template: '<div>step1-content</div>' },
        },
        {
            key: 'step2',
            label: 'step2-label',
            component: { template: '<div>step2-content</div>' },
        },
        {
            key: 'step3',
            label: 'step3-label',
            component: { template: '<div>step3-content</div>' },
        },
    ];
    const propsData: Partial<PageStepsProps> = {
        title: 'title',
        stepOptions: options,
    };

    // act -> mount 组件
    cy.mount(PageSteps, {
        propsData,
    });

    // Assert -> 断言标题是否与传入的 props 一致
    cy.contains('.sf-layout__title', 'title');
    // 断言渲染的步骤条是否正常
    cy.vStep_getSteps(SELECTOR.step).should('have.length', 3);
    cy.vStep_isActiveStep(SELECTOR.step, 'step1-label');
    cy.get(SELECTOR.prev).should('not.exist');
    cy.contains(SELECTOR.next, '下一步');
    cy.contains(SELECTOR.cancel, '取消');

    // 重复部分 Act -> Assert 操作
    cy.vButton_click(SELECTOR.next);
    cy.vStep_isActiveStep(SELECTOR.step, 'step2-label');
    cy.vButton_click(SELECTOR.prev);
    cy.vStep_isActiveStep(SELECTOR.step, 'step1-label');

    cy.vButton_click(SELECTOR.next);
    cy.vButton_click(SELECTOR.next);
    cy.get(SELECTOR.next).should('not.exist');
    cy.contains(SELECTOR.complete, '完成');
});
```



#### 如何做业务模块的测试

模拟真正用户使用这个系统的行为

1. 访问页面
2. 进行一些操作
3. 断言

```ts
describe('Td', () => {
    beforeEach(() => {
        cy.loginByPage({ name: 'admin', password: 'Test@123' });
    });

    it('Td', () => {
        const TABLE = '.sf-layout__tree-container-content .sfis-grid';
     	const text = 'wym-disk';

        cy.intercept('/admin/view/server-list*').as('getVMList');
        cy.visit(`/#/mod-computer/index?vm-page=VmMod`);
        cy.get('.sf-layout-main').should('be.visible');
        cy.get('[utid=vm-toolbar] [utid=search]')
            .type('wym-disk{enter}')
            .wait('@getVMList')
            .vTable_getRow(TABLE, { text })
            .vButton_click('[utid=more]')
            .vButton_click('[utid=grid-menu] [utid=edit]')
            .vModal_isVisible('[utid=single-edit-vm-win]')
            .vMask_wait('[utid=single-edit-vm-win]')
            .vTab_selectTab('.single-edit-vm-win_tab', '配置与网络')
            .vInput_containTip('[utid=disk-size]', '快速派生镜像生成的云主机不支持编辑磁盘大小');
    });
});
```
