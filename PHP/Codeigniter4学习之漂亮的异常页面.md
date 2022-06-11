### PHP
### 起因

CodeIgniter一直是我喜欢的一个框架，因为他轻量小巧，可是随着环境的变化，这个框架没有及时的更新，似乎有些没有赶上时代，但因为他小巧轻量的特点，仍然有无数的开发者在使用。

据说Codeigniter4出来了，引入命名空间，整个框架都脱胎换骨，这让我很兴奋，于是根据官方文档，上去就写了个hello world程序，发现框架提供了漂亮的异常页面，exception抛出来可以理解，fatal error和parse error的语法错误都可以提示出来，哎呀，像发现了新大陆一样兴奋，带着喜欢和好奇感赶紧研究了一番，研究之后就想据为己有，于是就想把这个异常抛出的操作提取出来，一方面是为了研究学习，一方面是想更灵活一些，在任何项目中都可以使用

### 提取完整代码

JsonExceptionHandler.php文件代码

```php
<?php

namespace Json\Exception;

use function error_reporting;
use ErrorException;
use Throwable;

/**
 * Exceptions manager
 */
class JsonExceptionHandler
{

	const EXIT_SUCCESS 		= 	0;		// no errors
	const EXIT_ERROR 		= 	1;		// generic error
	const EXIT__AUTO_MIN 	= 	9;		// lowest automatically-assigned error code
	const EXIT__AUTO_MAX 	= 	125;	// highest automatically-assigned error code

						
	use RenderTrait;

	public static function register() 
	{
		$_this = new self();
		$_this->initialize();
	}


	/**
	 * Responsible for registering the error, exception and shutdown
	 * handling of our application.
	 */
	public function initialize()
	{	
		error_reporting(E_ALL);

		//Set the Exception Handler
		set_exception_handler([$this, 'exceptionHandler']);

		// Set the Error Handler
		set_error_handler([$this, 'errorHandler']);

		// Set the handler for shutdown to catch Parse errors
		// Do we need this in PHP7?
		register_shutdown_function([$this, 'shutdownHandler']);
	}

	//--------------------------------------------------------------------

	/**
	 * Catches any uncaught errors and exceptions, including most Fatal errors
	 * (Yay PHP7!). Will log the error, display it if display_errors is on,
	 * and fire an event that allows custom actions to be taken at this point.
	 *
	 * @param \Throwable $exception
	 */
	public function exceptionHandler(Throwable $exception)
	{
		// @codeCoverageIgnoreStart
		$codes      = $this->determineCodes($exception);
		$statusCode = $codes[0];
		$exitCode   = $codes[1];

		$this->render($exception, $statusCode);

		exit($exitCode);
	}

	//--------------------------------------------------------------------

	/**
	 * Even in PHP7, some errors make it through to the errorHandler, so
	 * convert these to Exceptions and let the exception handler log it and
	 * display it.
	 *
	 * This seems to be primarily when a user triggers it with trigger_error().
	 *
	 * @param integer      $severity
	 * @param string       $message
	 * @param string|null  $file
	 * @param integer|null $line
	 *
	 * @throws \ErrorException
	 */
	public function errorHandler(int $severity, string $message, string $file = null, int $line = null)
	{
		if (! (error_reporting() & $severity))
		{
			return;
		}

		// Convert it to an exception and pass it along.
		throw new ErrorException($message, 0, $severity, $file, $line);
	}

	//--------------------------------------------------------------------

	/**
	 * Checks to see if any errors have happened during shutdown that
	 * need to be caught and handle them.
	 */
	public function shutdownHandler()
	{
		$error = error_get_last();

		// If we've got an error that hasn't been displayed, then convert
		// it to an Exception and use the Exception handler to display it
		// to the user.
		if (! is_null($error))
		{
			// Fatal Error?
			if (in_array($error['type'], [E_ERROR, E_CORE_ERROR, E_COMPILE_ERROR, E_PARSE]))
			{
				$this->exceptionHandler(new ErrorException($error['message'], $error['type'], 0, $error['file'], $error['line']));
			}
		}
	}


	//--------------------------------------------------------------------

	/**
	 * Gathers the variables that will be made available to the view.
	 *
	 * @param \Throwable $exception
	 * @param integer    $statusCode
	 *
	 * @return array
	 */
	protected function collectVars(Throwable $exception, int $statusCode): array
	{
		return [
			'title'   => get_class($exception),
			'type'    => get_class($exception),
			'code'    => $statusCode,
			'message' => $exception->getMessage() ?? '(null)',
			'file'    => $exception->getFile(),
			'line'    => $exception->getLine(),
			'trace'   => $exception->getTrace(),
		];
	}

	/**
	 * Determines the HTTP status code and the exit status code for this request.
	 *
	 * @param \Throwable $exception
	 *
	 * @return array
	 */
	protected function determineCodes(Throwable $exception): array
	{
		$statusCode = abs($exception->getCode());

		if ($statusCode < 100 || $statusCode > 599)
		{
			$exitStatus = $statusCode + self::EXIT__AUTO_MIN; // 9 is EXIT__AUTO_MIN
			if ($exitStatus > self::EXIT__AUTO_MAX) // 125 is EXIT__AUTO_MAX
			{
				$exitStatus = self::EXIT_ERROR; // EXIT_ERROR
			}
			$statusCode = 500;
		}
		else
		{
			$exitStatus = 1; // EXIT_ERROR
		}

		return [
			$statusCode ?? 500,
			$exitStatus,
		];
	}

	//--------------------------------------------------------------------
	//--------------------------------------------------------------------
	// Display Methods
	//--------------------------------------------------------------------

	/**
	 * Clean Path
	 *
	 * This makes nicer looking paths for the error output.
	 *
	 * @param string $file
	 *
	 * @return string
	 */
	public static function cleanPath(string $file): string
	{
		return $file;
	}


	//--------------------------------------------------------------------

	/**
	 * Creates a syntax-highlighted version of a PHP file.
	 *
	 * @param string  $file
	 * @param integer $lineNumber
	 * @param integer $lines
	 *
	 * @return boolean|string
	 */
	public static function highlightFile(string $file, int $lineNumber, int $lines = 15)
	{
		if (empty($file) || ! is_readable($file))
		{
			return false;
		}

		// Set our highlight colors:
		if (function_exists('ini_set'))
		{
			ini_set('highlight.comment', '#767a7e; font-style: italic');
			ini_set('highlight.default', '#c7c7c7');
			ini_set('highlight.html', '#06B');
			ini_set('highlight.keyword', '#f1ce61;');
			ini_set('highlight.string', '#869d6a');
		}

		try
		{
			$source = file_get_contents($file);
		}
		catch (Throwable $e)
		{
			return false;
		}

		$source = str_replace(["\r\n", "\r"], "\n", $source);
		$source = explode("\n", highlight_string($source, true));
		$source = str_replace('<br />', "\n", $source[1]);

		$source = explode("\n", str_replace("\r\n", "\n", $source));

		// Get just the part to show
		$start = $lineNumber - (int) round($lines / 2);
		$start = $start < 0 ? 0 : $start;

		// Get just the lines we need to display, while keeping line numbers...
		$source = array_splice($source, $start, $lines, true);

		// Used to format the line number in the source
		$format = '% ' . strlen(sprintf('%s', $start + $lines)) . 'd';

		$out = '';
		// Because the highlighting may have an uneven number
		// of open and close span tags on one line, we need
		// to ensure we can close them all to get the lines
		// showing correctly.
		$spans = 1;

		foreach ($source as $n => $row)
		{
			$spans += substr_count($row, '<span') - substr_count($row, '</span');
			$row    = str_replace(["\r", "\n"], ['', ''], $row);

			if (($n + $start + 1) === $lineNumber)
			{
				preg_match_all('#<[^>]+>#', $row, $tags);
				$out .= sprintf("<span class='line highlight'><span class='number'>{$format}</span> %s\n</span>%s", $n + $start + 1, strip_tags($row), implode('', $tags[0])
				);
			}
			else
			{
				$out .= sprintf('<span class="line"><span class="number">' . $format . '</span> %s', $n + $start + 1, $row) . "\n";
			}
		}

		if ($spans > 0)
		{
			$out .= str_repeat('</span>', $spans);
		}

		return '<pre><code>' . $out . '</code></pre>';
	}

}


trait RenderTrait
{
	/**
	 * Given an exception and status code will display the error to the client.
	 *
	 * @param \Throwable $exception
	 * @param integer    $statusCode
	 */
	protected function render(Throwable $exception, int $statusCode)
	{
		
		// Prepare the vars
		$vars = $this->collectVars($exception, $statusCode);
		extract($vars);

		ob_start();


		$error_id = uniqid('error', true);
		echo	'<!doctype html>';
		echo	'<html>';
		echo 	'<head>';
		echo 		'<meta charset="UTF-8">';
		echo 		'<meta name="robots" content="noindex">';
		echo 		'<title>'.htmlspecialchars($title, ENT_SUBSTITUTE, 'UTF-8').'</title>';
		echo 		'<style type="text/css">';
		$css = '
			body {
			    height: 100%;
			    background: #fafafa;
			    font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
			    color: #777;
			    font-weight: 300;
			    margin: 0;
			    padding: 0;
			}
			h1 {
			    font-weight: lighter;
			    letter-spacing: 0.8;
			    font-size: 3rem;
			    color: #222;
			    margin: 0;
			}
			h1.headline {
			    margin-top: 20%;
			    font-size: 5rem;
			}
			.text-center {
			    text-align: center;
			}
			p.lead {
			    font-size: 1.6rem;
			}
			.container {
			    max-width: 75rem;
			    margin: 0 auto;
			    padding: 1rem;
			}
			.header {
			    background: #85271f;
			    color: #fff;
			}
			.header h1 {
			    color: #fff;
			}
			.header p {
			    font-size: 1.2rem;
			    margin: 0;
			    line-height: 2.5;
			}
			.header a {
			    color: rgba(255,255,255,0.5);
			    margin-left: 2rem;
			    display: none;
			    text-decoration: none;
			}
			.header:hover a {
			    display: inline;
			}

			.footer .container {
			    border-top: 1px solid #e7e7e7;
			    margin-top: 1rem;
			    text-align: center;
			}

			.source {
			    background: #333;
			    color: #c7c7c7;
			    padding: 0.5em 1em;
			    border-radius: 5px;
			    font-family: Menlo, Monaco, Consolas, "Courier New", monospace;
			    margin: 0;
			}
			.source span.line {
			    line-height: 1.4;
			}
			.source span.line .number {
			    color: #666;
			}
			.source .line .highlight {
			    display: block;
			    background: #555;
			    color: #fff;
			}
			.source span.highlight .number {
			    color: #fff;
			}

			.tabs {
			    list-style: none;
			    list-style-position: inside;
			    margin: 0;
			    padding: 0;
			    margin-bottom: -1px;
			}
			.tabs li {
			    display: inline;
			}
			.tabs a:link,
			.tabs a:visited {
			    padding: 0rem 1rem;
			    line-height: 2.7;
			    text-decoration: none;
			    color: #a7a7a7;
			    background: #f1f1f1;
			    border: 1px solid #e7e7e7;
			    border-bottom: 0;
			    border-top-left-radius: 5px;
			    border-top-right-radius: 5px;
			    display: inline-block;
			}
			.tabs a:hover {
			    background: #e7e7e7;
			    border-color: #e1e1e1;
			}
			.tabs a.active {
			    background: #fff;
			}
			.tab-content {
			    background: #fff;
			    border: 1px solid #efefef;
			}
			.content {
			    padding: 1rem;
			}
			.hide {
			    display: none;
			}

			.alert {
			    margin-top: 2rem;
			    display: block;
			    text-align: center;
			    line-height: 3.0;
			    background: #d9edf7;
			    border: 1px solid #bcdff1;
			    border-radius: 5px;
			    color: #31708f;
			}
			ul, ol {
			    line-height: 1.8;
			}

			table {
			    width: 100%;
			    overflow: hidden;
			}
			th {
			    text-align: left;
			    border-bottom: 1px solid #e7e7e7;
			    padding-bottom: 0.5rem;
			}
			td {
			    padding: 0.2rem 0.5rem 0.2rem 0;
			}
			tr:hover td {
			    background: #f1f1f1;
			}
			td pre {
			    white-space: pre-wrap;
			}

			.trace a {
			    color: inherit;
			}
			.trace table {
			    width: auto;
			}
			.trace tr td:first-child {
			    min-width: 5em;
			    font-weight: bold;
			}
			.trace td {
			    background: #e7e7e7;
			    padding: 0 1rem;
			}
			.trace td pre {
			    margin: 0;
			}
			.args {
			    display: none;
			}
		';
		echo 	preg_replace('#[\r\n\t ]+#', ' ', $css);	
		echo 		'</style>';
		echo 	'</head>';

		echo 	'<body>';

		echo 		'<div class="header">';
		echo 			'<div class="container">';
		echo 				'<h1>'.htmlspecialchars($title, ENT_SUBSTITUTE, 'UTF-8'). ($exception->getCode() ? ' #' . $exception->getCode() : '').'</h1>';
		echo 				'<p>'.$exception->getMessage().'</p>';
		echo			'</div>';
		echo 		'</div>';

		echo 		'<div class="container">';
		echo 			'<p><b>'.static::cleanPath($file, $line).'</b> at line <b>'. $line.'</b></p>';

		if (is_file($file)) {
			echo 		'<div class="source">';
			echo 		static::highlightFile($file, $line, 15);
			echo 		'</div>';
		}
		echo 		'</div>';

		echo		'<div class="container">';

		echo 			'<ul class="tabs" id="tabs">';
		echo 				'<li><a href="#backtrace">Backtrace</a></li>';	
		echo 			'</ul>';
		echo 			'<div class="tab-content">';

		echo 				'<div class="content" id="backtrace">';

		echo					'<ol class="trace">';
									foreach ($trace as $index => $row) {
		echo						'<li>';
		echo							'<p>';
											//Trace info
											if (isset($row['file']) && is_file($row['file'])){
												if (isset($row['function']) && in_array($row['function'], ['include', 'include_once', 'require', 'require_once']))
												{
		echo 										$row['function'] .' ' . static::cleanPath($row['file']);
												}
												else
												{
		echo 										static::cleanPath($row['file']).' : '.$row['line'];
												}
											}else {
		echo 									'{PHP internal code}';
											}

											//Class/Method
											if (isset($row['class'])) {
		echo									'&nbsp;&nbsp;&mdash;&nbsp;&nbsp;'.$row['class'] . $row['type'] . $row['function'];
												if (! empty($row['args'])) {
													$args_id = $error_id . 'args' . $index;
		echo										'( <a href="#" onclick="return toggle(\''.$args_id.'\');">arguments</a> )';
		echo										'<div class="args" id="'.$args_id.'">';
		echo											'<table cellspacing="0">';

														$params = null;
														// Reflection by name is not available for closure function
														if (substr( $row['function'], -1 ) !== '}')
														{
															$mirror = isset( $row['class'] ) ? new \ReflectionMethod( $row['class'], $row['function'] ) : new \ReflectionFunction( $row['function'] );
															$params = $mirror->getParameters();
														}
														foreach ($row['args'] as $key => $value) {
		echo												'<tr>';
		echo													'<td><code>'. htmlspecialchars(isset($params[$key]) ? '$' . $params[$key]->name : "#$key", ENT_SUBSTITUTE, 'UTF-8').'</code></td>';
		echo													'<td><pre>'. print_r($value, true) .'</pre></td>';
		echo												'</tr>';
														}
		echo											'</table>';
		echo										'</div>';
												} else{
		echo										'()';
												}
											}

											if (! isset($row['class']) && isset($row['function'])) {
		echo									'&nbsp;&nbsp;&mdash;&nbsp;&nbsp;'.$row['function'].'()';
											}
		echo							'</p>';

										if (isset($row['file']) && is_file($row['file']) &&  isset($row['class'])) {
		echo								'<div class="source">';
		echo 									static::highlightFile($row['file'], $row['line']);
		echo								'</div>';
										}
		echo 						'</li>';
									}
		echo					'</ol>';
		echo 				'</div>';
		echo 			'</div>';
		echo 		'</div>';

		echo 		'<div class="footer">';
		echo 			'<div class="container">';
		echo 				'<p>';
		echo					'Displayed at '.date('H:i:sa').' &mdash;';
		echo					'PHP:'.phpversion();
		echo 				'</p>';

		echo 			'</div>';
		echo 		'</div>';

		echo 	'</body>';
		echo 	'</html>';

		$buffer = ob_get_contents();
		ob_end_clean();
		echo $buffer;
	}
}
```

### 使用方法
提取出来之后，只需要一个文件，只需要三步，就可以使用了

1. 引入文件
2. 使用命名空间下的类
3. 注册

```php
<?php
use Json\Exception\JsonExceptionHandler;

require './JsonExceptionHandler.php';//引入文件

JsonExceptionHandler::register();//注册异常、错误处理函数

//调用一个不存在的函数
test();
```
### 异常效果图
![exception.jpg](/usr/uploads/2020/07/69169028.jpg)

是不是很漂亮，赶紧在自己的电脑上试试吧,**只需要把上面的源码复制到自己的项目文件里面就可用啦**

### 其它
如果你项目中使用composer，可以使用composer进行安装和使用
```shell
composer require fanqingxuan/exception
```

```php
<?php

require 'vendor/autoload.php';//引入文件

use Json\Exception\JsonExceptionHandler;

JsonExceptionHandler::register();//注册异常、错误处理函数

//调用一个不存在的函数
var_dump(test());
```