# 개요

D3는 차트 라이브러리가 아니다. 차트를 그릴 수는 있지만,

참고 http://www.macwright.org/presentations/dcjq/

D3는 데이터를 화면 구성 요소에 연결시켜서 데이터를 다양한 방식으로 볼 수 있게 해주는 시각화 라이브러리다. 


# Selection

CSS Selector로 선택한 DOM 요소의 D3 기능 Wrapper 객체
Selection은 2차원 배열이고, d3_selectionPrototype.each 메서드를 통해 2차원 배열 내 각 요소에 대해 동작

## d3.select(selector)

`document.querySelector(selector)`로 반환 받은 node에 []를 씌워 1차원 배열로 만들고, 그 배열에 []를 한 번 더 씌워 2차원 배열로 만들고, 그 2차원 배열의 `__proto__`에 d3.selectionPrototype을 할당해서 d3의 selection이 가진 기능들을 추가하고, 그 2차원 배열을 반환한다.

959라인

    d3.select = function(node) {
        var group = [document.querySelector(node) 아니면 node]; // node가 문자열이면 querySelector, 아니면 node
        group.parentNode = document.documentElement; // <html> 임
        return d3_selection([group]);
        // return d3_selection([group])는 결국 아래와 같다.
        // [group].__proto__ = d3.selection.prototype; // d3.selection.prototype을 상속
        // return [group]; // group은 1차원 배열, [group]은 2차원 배열        
    };

반환되는 것은 `[0][0]`인 2차원 배열이다.

469라인의 `d3_selection(groups)`은 `d3_subclass(groups, d3_selectionPrototype)`를 호출해서 `groups.__proto__ = d3.selection.prototype`을 실행해서 d3의 selection의 기능을 추가해준다. d3의 selection의 기능은 `d3_selectionPrototype.` 으로 검색하면 됨.

실제 d3.js 코드에서는 `d3_selectionPrototype`은 `d3.selection.prototype`과 같으며, 모든 selection은 이 prototype을 통해 d3의 selection의 기능을 물려받음

d3.select의 group은 단순 리터럴 배열이다.

## d3.selectAll(selector)

`document.querySelectorAll(selector)`로 반환 받은 유사 배열 객체인 NodeList를 일반 1차원 배열로 변환하고, []를 씌워서 2차원 배열을 만들고, 그 2차원 배열의 `__proto__`에 `d3.selection.prototype`을 할당해서 d3의 selection이 가진 기능들을 추가하고, 그 2차원 배열을 반환한다.

964 라인

    d3.selectAll = function(nodes) {
        var group = d3_array(typeof nodes === "string" ? d3_selectAll(nodes, d3_document) : nodes); // d3_selectAll(a, b)는 b.querySelectorAll(a)을 실행한다. 다른 일은 안함.
        group.parentNode = d3_documentElement;
        return d3_selection([ group ]);
    };

`d3_selectionPrototype`을 `[group].__proto__`에 하당하는 것은 `d3.select`와 같다.

`d3.select`와 다른 점은 반환되는 객체가 `[0][0]`이 아닌 `[0][n]`라는 점이다.

377~385라인
`d3_array(arg)`는 `Array.prototype.slice.call(arg)`를 통해 querySelectorAll()로 반환된 유사 배열인 NodeList를 JavaScript의 일반 배열로 변환하여 반환. 다른 처리는 없다.

## d3_selection(groups)
groups는 노드를 원소로 하는 그룹 배열의 그룹이다. 즉, groups는 2차원 배열이고, groups에 d3_selectionPrototype을 심어서 셀렉션이 된다.  

```` javascript
function d3_selection(groups) {
    d3_subclass(groups, d3_selectionPrototype);
    return groups;
}
````

## d3_subclass(object, prorotype)

__proto__를 지원하면 __proto__에 d3_selectionPrototype을 할당하고,
__proto__를 지원하지 않으면, d3_selectionPrototype의 모든 속성을 object의 속성으로 추가해서 object에게 d3의 기본 기능을 심어준다. object는 노드를 원소로 하는 그룹 배열의 배열, 즉 2차원 배열이다.

```` javascript
var d3_subclass = {}.__proto__ ? function(object, prototype) {
    object.__proto__ = prototype;
} : function(object, prototype) {
    for (var property in prototype) object[property] = prototype[property];
};
````



## selection.select(selector)

493라인

selection은 이미 선택된 parentSelection
subgroups는 반환될 newSelection
group는 parentSelection 1차원째 원소인 배열, group===selection[n]
node는 parentSelection 2차원째 원소인 요소, node===selection[n][n]
subgroup는 subgroups의 1차원째 원소인 배열, subgroup===subgroups[n]
subnode는 subgroups의 2차원째 원소인 요소, subnode===subgroups[n][n]


selection.select는 subgroup의 parentNode에 원래 group에 있던 parentNode를 할당한다.
즉, 반환되는 selection의 group이 바뀌지 않는다.


## selection.selectAll(selector)

515라인

selection은 이미 선택된 parentSelection
subgroups는 반환될 newSelection
group는 parentSelection 1차원째 원소인 배열, group===selection[n]
node는 parentSelection 2차원째 원소인 요소, node===selection[n][n]
subgroup는 subgroups의 1차원째 원소인 배열, subgroup===subgroups[n]

selection.selectAll은 subgroup의 parentNode에 node를 할당한다.
즉, 반환되는 selection의 group이 바뀐다.

## selection의 동작

D3의 selection은 2차원 배열이고,
863~875와 같이 each 메서드를 통해 2차원 배열의 각 요소에 대해 callback.call(node, node.__data__, i, j) 즉, node.callback(d, i, j)와 같이 동작한다.

위 방식의 유일한 예외는 selection.data 메서드다.

# Data

data() 함수를 호출한 selection의 말단 HTML Element에 `__data__` property를 추가하여 거기에 데이터를 넣어 DOM과 데이터를 Join 한다.

datum() 함수는 819라인에 있는데, 해당 셀렉션에 있는 각 문서요소에 __data__ = value를 해주는 것이 전부이다. 즉, parentNode, update, enter, exit와 무관하게 단순히 값을 할당하는 것 뿐이다.  

## selection.data(value, key)

732~813

### 아무 args가 없으면
group은 selection의 group
node는 selection의 말단 HTML Element
value는 단순 배열이며 value 배열의 각 요소는 node의 `__data__`에 저장된 값을 할당받는다.
결국 selection의 말단 Element노드에 들어있는 값으로 구성된 배열을 반환한다.

### args 가 있으면 bind(group, groupData)

743~795
group은 selection의 1차원째 원소인 배열
groupData는
  value가 함수이면 함수.call(group, group.parentNode.__data__, i)가 반환하는 배열
  value가 배열이면 그 배열

group[i]가 존재하지 않고 groupData[i]가 있으면, 
  {'__data__' : groupData[i]}를 enter 배열에 추가
group[i]가 존재하고 groupData[i]도 존재하면,
  group[i].__data__에 groupData[i]를 할당하고,
  update 배열에 group[i]를 추가
group[i]가 존재하고, groupData[i]가 존재하지 않으면 group[i]를 exit 배열에 추가

enter, update, exit 배열은 d3_selection 함수에 의해 prototype에 d3_selectionPrototype이 할당된 d3 selection 객체다.

update selection은 enter selection과 exit selection을 반환하는 함수를 포함하고 있고,

data 함수는 결국 update selection을 반환한다.

# 메모들
--

## 데이터 바인딩 vs 데이터 조인

### 데이터 바인딩

D3에서의 데이터 바인딩은 어떤 데이터를 문서요소의 __data__ 속성에 값으로 할당하는 것을 의미하며 기본적으로 값:값의 관계다.

### 데이터 조인

이에 반해 데이터 조인은, 데이터 배열을 셀렉션의 그룹 배열에 어떤 규칙을 가지고 접목하는 것으로 호스트 코드 상에서는 배열:배열의 관계다.

### 둘의 관계

데이터 조인은 데이터 바인딩의 한 방식이며, 데이터 조인을 하면 결과적으로 데이터 바인딩이 실행된다.

## 셀렉션, 그룹, 노드

### 셀렉션

### 그룹

### 노드

## Declaritive

http://bost.ocks.org/mike/join/

d3에게 어떻게 하라고 알려주는 대신, 니가 원하는 게 뭔지 알려줘라.

var circle = svg.selectAll('ddd')
                .data(data);

circle.exit().remove();
circle.enter().append('circle').attr('r', 2.5);
circle.attr('cx', function(d) { return d.x; })
      .attr('cy', function(d) { return d.y; });

if나 데이터와 조인된 것, 조인되지 않은 것을 걸러내서 처리하고,
for나 while 없이도 여러 개의 원을 그린다.
