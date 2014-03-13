# Şablonlar

- [Denetçi (Controller) Düzenleri](#controller-layouts)
- [Blade Şablonları](#blade-templating)
- [Diğer Blade Kontrol Yapıları](#other-blade-control-structures)
- [Extending Blade](#extending-blade)

<a name="controller-layouts"></a>
## Denetçi (Controller) Düzenleri

Laravel'de şablon kullanma yöntemlerinden birisi denetçi düzenleri üzerinden gerçekleştirilir. İlgili denetçideki `layout` özelliğinin belirlenmesiyle, belirlemiş olduğunuz görünüm oluşturulacak ve eylemlerden dönmüş cevap olarak kabul edilecektir.

#### Bir Denetçide Bir Düzen Tanımlanması

	class UyeController extends BaseController {

		/**
		 * Cevaplar için kullanılacak olan düzen.
		 */
		protected $layout = 'layouts.master';

		/**
		 * Uye profilini göster.
		 */
		public function showProfil()
		{
			$this->layout->content = View::make('uye.profil');
		}

	}

<a name="blade-templating"></a>
## Blade Şablonları

Blade Laravel'le gelen basit ama güçlü bir şablon motorudur. Denetçi düzenlerinden farklı olarak, Blade _şablon kalıtımı_ ve _kesimler_ (sections) ile yürütülür. Tüm Blade şablonlarının uzantısı `.blade.php` olmalıdır.

#### Bir Blade Düzeninin Tanımlanması

	<!-- app/views/layouts/master.blade.php 'de bulunmaktadır-->

	<html>
		<body>
			@section('sidebar')
				This is the master sidebar.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

#### Bir Blade Düzeninin Kullanılması

	@extends('layouts.master')

	@section('sidebar')
		@parent

		<p>Burası master sidebar'a eklenmiştir.</p>
	@stop

	@section('content')
		<p>Burası kendi content bölümümdür.</p>
	@stop

Bir Blade düzenini genişleten (`extend`) görünümlerin, düzenden gelen kesimleri değiştirmekten başka bir şey yapmadığını unutmayın. İlgili düzenin içeriği bir kesimde `@parent` direktifi kullanılarak çocuk görünüme katılabilir, böylece bir kenar çubuğu veya altbilgi gibi bir düzen kesimine eklemeler yapabilirsiniz.

Kimi zaman, örneğin bir kesimin tanımlanmış olup olmadığından emin olmadığınız durumlarda, `@yield` direktifine ön tanımlı bir değer geçmek isteyebilirsiniz. Bu ön tanımlı değer ikinci parametre olarak geçilebilir.

	@yield('section', 'Ön Tanımlı İçerik');

<a name="other-blade-control-structures"></a>
## Diğer Blade Kontrol Yapıları

#### Veri Yazdırılması

	Merhaba {{ $isim }}.

	Şu anki UNIX zaman damgası {{ time() }}'dır.

Küme parantezleri ile sarmalanmış bir metni görüntülemek isterseniz, küme parantezi önüne `@` sembolü ilave ederek Blade davranışını devredışı bırakabilirsiniz.

#### Küme Parantezi İle Ham Metin Görüntülemek

	@{{ Bu metin Blade tarafından işleme alınmayacaktır }}

Tabii ki, kullanıcılardan gelen tüm veriler escape edilmeli ya da arındırılmalıdır. Çıktıyı escape etmek için, üçlü küme parantezi sözdizimini kullanabilirsiniz:

	Merhaba {{{ $isim }}}.

> **Not:** Uygulamanızın kullanıcılarından gelen verileri yazdıracağınız zaman çok dikkatli olun. İçerikte olabilecek HTML antitelerini escape etmek amacıyla her zaman için üçlü küme parantezi sözdizimi kullanın.

#### If Cümleleri

	@if (count($records) === 1)
		Tek kayıt var!
	@elseif (count($records) > 1)
		Birden çok kayıt var!
	@else
		Hiç kayıt yok!
	@endif

	@unless (Auth::check())
		Giriş yapmadınız.
	@endunless

#### Döngüler

	@for ($i = 0; $i < 10; $i++)
		Şu anki değer {{ $i }}'dir.
	@endfor

	@foreach ($uyeler as $uye)
		<p>Bu, üye {{ $uye->id }}'dir.</p>
	@endforeach

	@while (true)
		<p>Sonsuz döngüdeyim.</p>
	@endwhile

#### Alt Görünümlerin Dahil Edilmesi

	@include('view.ismi')

Dahil edilen görünüme bir veri dizisi de geçebilirsiniz:

	@include('view.ismi', array('birsey'=>'veri'))

#### Kesimlerin Üzerine Yazmak

Ön tanımlı olarak, section'lar sectionda önceden mevcut olan bir içeriğe eklenirler. Bir section'u, öncekini geçersiz kılarak tümden üzerine yazmak için `overwrite` cümlesini kullanabilirsiniz:

	@extends('list.item.container')

	@section('list.item.content')
	    <p>Bu {{ $item->type }} tipinde bir öğedir</p>
	@overwrite

#### Dil Satırlarının Gösterilmesi

	@lang('language.line')

	@choice('language.line', 1);

#### Yorumlar

	{{-- Bu yorum, gösterilen HTML içerisinde olmayacaktır --}}

<a name="extending-blade"></a>
## Extending Blade

Blade even allows you to define your own custom control structures. When a Blade file is compiled, each custom extension is called with the view contents, allowing you to do anything from simple `str_replace` manipulations to more complex regular expressions.

The Blade compiler comes with the helper methods `createMatcher` and `createPlainMatcher`, which generate the expression you need to build your own custom directives.

The `createPlainMatcher` method is used for directives with no arguments like `@endif` and `@stop`, while `createMatcher` is used for directives with arguments.

The following example creates a `@datetime($var)` directive which simply calls `->format()` on `$var`:

	Blade::extend(function($view, $compiler)
	{
		$pattern = $compiler->createMatcher('datetime');

		return preg_replace($pattern, '$1<?php echo $2->format('m/d/Y H:i'); ?>', $view);
	});