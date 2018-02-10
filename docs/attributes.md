<!-- # Attributes -->
# 属性

<!-- ## Common Sources -->
## 一般的なソース

<!-- Attribute sources are used to define the strategy by which block attribute values are extracted from saved post content. They provide a mechanism to map from the saved markup to a JavaScript representation of a block. -->
属性ソースは、保存された投稿コンテンツからどのブロック属性値が抽出されたのかによって方策を決定するのに使用されます。保存されたマークアップをひとつのブロックとしてのひとつのJavaScriptの表現にマップするメカニズムを提供します。

<!-- Each source accepts an optional selector as the first argument. If a selector is specified, the source behavior will be run against the corresponding element(s) contained within the block. Otherwise it will be run against the block's root node. -->
それぞれのソースは最初の引数として任意のセレクターを受けとります。セレクターが指定されると、そのソースはそのブロック内に含まれる対応する要素に対して動作します。さもなければ、そのブロックのルートノードに対して動作します。

<!-- Under the hood, attribute sources are a superset of functionality provided by [hpq](https://github.com/aduth/hpq), a small library used to parse and query HTML markup into an object shape. In an object of attributes sources, you can name the keys as you see fit. The resulting object will assign as a value to each key the result of its attribute source. -->
内部は、属性ソースは[hpq](https://github.com/aduth/hpq)によって提供される機能のスーパーセットです。[hpq](https://github.com/aduth/hpq)は、HTMLマークアップをパースしクエリーによってオブジェクト形式に変換する小さなライブラリーです。属性ソースのオブジェクト内で参照したいキーを名前付けすることができます。結果として生じるオブジェクトは、その属性ソースの結果の各キーに値としてアサインされます。

### `attribute`

<!-- Use `attribute` to extract the value of an attribute from markup. -->
 `attribute` を使用してマークアップから属性の値を抽出します。

<!-- _Example_: Extract the `src` attribute from an image found in the block's markup. -->
_例_: ブロックのマークアップ内に見つかった画像から `src` 属性を抽出します。

```js
{
	url: {
		source: 'attribute',
		selector: 'img',
		attribute: 'src',
	}
}
// { "url": "https://lorempixel.com/1200/800/" }
```

### `text`

<!-- Use `text` to extract the inner text from markup. -->
`text` を使用してマークアップから内側のテキストを抽出します。

```js
{
	content: {
		source: 'text',
		selector: 'figcaption',
	}
}
// { "content": "The inner text of the figcaption element" }
```

### `html`

<!-- Use `html` to extract the inner HTML from markup. -->
`html` を使用してマークアップから内側のHTMLを抽出します。

```js
{
	content: {
		source: 'html',
		selector: 'figcaption',
	}
}
// { "content": "The inner text of the <strong>figcaption</strong> element" }
```

### `children`

<!-- Use `children` to extract child nodes of the matched element, returned as an array of virtual elements. This is most commonly used in combination with the `RichText` component. -->
`children` を使用してマッチした要素の子ノードを抽出し、仮想要素の配列として返します。これは大抵の場合 `RichText` コンポーネントと組み合わせて利用されます。

<!-- _Example_: Extract child nodes from a paragraph of rich text. -->
_例_: リッチテキストのパラグラフから子ノードを抽出します。

```js
{
	content: {
		source: 'children',
		selector: 'p'
	}
}
// {
//   "content": [
//     "Vestibulum eu ",
//     { "type": "strong", "children": "tortor" },
//     " vel urna."
//   ]
// }
```

### `query`

<!-- Use `query` to extract an array of values from markup. Entries of the array are determined by the selector argument, where each matched element within the block will have an entry structured corresponding to the second argument, an object of attribute sources. -->
`query` を使用してマークアップから値の配列を抽出します。配列のエントリーはセレクター引数によって決定され、ブロック内のそれぞれにマッチした要素は、要素ソースのオブジェクトである2番めの引数に対応したエントリーを持ちます。

<!-- _Example_: Extract `src` and `alt` from each image element in the block's markup. -->
_例_: ブロックのマークアップ内の各画像要素から `src` と `alt` を抽出します。

```js
{
	images: {
		source: 'query'
		selector: 'img',
		query: {
			url: { source: 'attribute', attribute: 'src' },
			alt: { source: 'attribute', attribute: 'alt' },
		}
	}
}
// {
//   "images": [
//     { "url": "https://lorempixel.com/1200/800/", "alt": "large image" },
//     { "url": "https://lorempixel.com/50/50/", "alt": "small image" }
//   ]
// }
```

## Meta

<!-- Attributes may be obtained from a post's meta rather than from the block's representation in saved post content. For this, an attribute is required to specify its corresponding meta key under the `meta` key: -->
属性は、保存された投稿コンテンツ内のブロックの表示部分ではなく投稿のメタから取得することもできます。このためには、 `meta` キー配下で対応するメタキーを指定するために属性が必要となります。

```js
attributes: {
	author: {
		type: 'string',
		source: 'meta',
		meta: 'author'
	},
},
```

<!-- From here, meta attributes can be read and written by a block using the same interface as any attribute: -->
ここから、どの属性としても同じインターフェイスを使うブロックによってメタ属性が読み書きできます。

{% codetabs %}
{% ES5 %}
```js
edit: function( props ) {
	function onChange( event ) {
		props.setAttributes( { author: event.target.value } );
	}

	return el( 'input', {
		value: props.attributes.author,
		onChange: onChange,
	} );
},
```
{% ESNext %}
```js
edit( { attributes, setAttributes } ) {
	function onChange( event ) {
		setAttributes( { author: event.target.value } );
	}

	return <input value={ attributes.author } onChange={ onChange } />;
},
```
{% end %}

<!-- ### Considerations -->
### 留意事項

<!-- By default, a meta field will be excluded from a post object's meta. This can be circumvented by explicitly making the field visible: -->
デフォルトではメタフィールドは投稿オブジェクトのメタからは除外されます。これはこのフィールドを明示的に可視化することにより避けられます。

```php
function gutenberg_my_block_init() {
	register_meta( 'post', 'author', array(
		'show_in_rest' => true,
	) );
}
add_action( 'init', 'gutenberg_my_block_init' );
```

<!-- Furthermore, be aware that WordPress defaults to: -->
さらに、WordPressでは以下がデフォルトとなっていることに気をつけてください:

<!-- - not treating a meta datum as being unique, instead returning an array of values; -->
- メタデータはひとつのものとしては扱われず、値の配列として返されます。
<!-- - treating that datum as a string. -->
- このデータは文字列として扱われます。

<!-- If either behavior is not desired, the same `register_meta` call can be complemented with the `single` and/or `type` parameters as follows: -->
どちらも望まない場合は、次のように同じ `register_meta` 呼び出しを `single` と/もしくは `type` パラメータで補完することができます:

```php
function gutenberg_my_block_init() {
	register_meta( 'post', 'author_count', array(
		'show_in_rest' => true,
		'single' => true,
		'type' => 'integer',
	) );
}
add_action( 'init', 'gutenberg_my_block_init' );
```

<!-- Lastly, make sure that you respect the data's type when setting attributes, as the framework does not automatically perform type casting of meta. Incorrect typing in block attributes will result in a post remaining dirty even after saving (_cf._ `isEditedPostDirty`, `hasEditedAttributes`). For instance, if `authorCount` is an integer, remember that event handlers may pass a different kind of data, thus the value should be cast explicitly: -->
最後に、このフレームワークはメタの型キャスティングを自動的には実行しないので、属性を設定する際にはデータのタイプに配慮するよう気をつけてください。ブロック属性内の誤った型は、保存後 (_cf._ `isEditedPostDirty`, `hasEditedAttributes`) でも投稿を望ましくない状態にしてしまいます。例えば、`authorCount` が整数値なら、ハンドラーでさえ違った種類のデータを渡すことがあり、したがってその値は明示的にキャストする必要があることを念頭に置いてください。

```js
function onChange( event ) {
	props.setAttributes( { authorCount: Number( event.target.value ) } );
}
```
