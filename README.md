<script src="phaser.js"></script>
<script type="text/javascript">

	var game = new Phaser.Game(480, 640, Phaser.AUTO, '', {preload: preload, create: create, update: update});

	var playerBet;
	var computerBet;
	var ball;

	var playerWin = 0;
	var computerWin = 0;

	var computerBetSpeed = 230;
	var ballSpeed = 300;
	var ballReleased = false;

	var level = 1;

	var hits = 0;

	function preload() {

		game.load.image('bet', 'assets/bet.png');
		game.load.image('ball', 'assets/ball.png');
		game.load.image('background', 'assets/starfield.jpg');
		game.load.image('star', 'assets/star.png');

	}


	function create() {
		//Скрываем курсор
		game.canvas.style.cursor = 'none';
		game.add.tileSprite(0, 0, 480, 640, 'background');
		playerBet = createBet(game.world.centerX, 600);
		computerBet = createBet(game.world.centerX, 20);
		textLevel = game.add.text(0, 0, 'Level: ' + level, {fill: '#efefef'})
		text = game.add.text(game.world.centerX, game.world.centerY, '', {fill: '#efefef'})
		text.anchor.setTo(0.5, 0.5);

		ball = game.add.sprite(game.world.centerX, game.world.centerY, 'ball');
		ball.anchor.setTo(0.5, 0.5);
		ball.body.collideWorldBounds = true;
		ball.body.bounce.setTo(1, 1);

		//Инициализируем эмиттер частиц
		emitter = game.add.emitter(0, 0, 200);
		emitter.makeParticles('star');

		game.input.onDown.add(releaseBall, this);
	}

	function particleBurst(x, y) {

		//Устанавливаем эмиттер частиц в точку x, y
		emitter.x = x;
		emitter.y = y;

		//Певрый паарметр определяет, нужно ли выпускать все чатсицы в один момент (режим  explode, взрыв)
		//Второй параметр устанавливает время жизни частицы в миллисекундах
		//В режиме burst/explode третий параметр игнорируется
		//Последний параметр определяет, сколько частиц будет выпущено при "взрыве"
		emitter.start(true, 500, null, 5);

	}

	function createBet(x, y) {
		var bet = game.add.sprite(x, y, 'bet');
		bet.anchor.setTo(0.5, 0.5);
		bet.body.collideWorldBounds = true;
		bet.body.bounce.setTo(1, 1);
		bet.body.immovable = true;

		return bet;
	}

	function update() {
		//Управляем ракеткой игрока
		playerBet.x = game.input.x;

		var playerBetHalfWidth = playerBet.width / 2;

		if (playerBet.x < playerBetHalfWidth) {
			playerBet.x = playerBetHalfWidth;
		}
		else if (playerBet.x > game.width - playerBetHalfWidth) {
			playerBet.x = game.width - playerBetHalfWidth;
		}
		//Управляем ракеткой компьютерного соперника
		if (computerBet.x - ball.x >= -15 && computerBet.x - ball.x <= 15) {
			computerBet.body.velocity.x = 0;
		}
		else if (computerBet.x - ball.x < -15) {
			computerBet.body.velocity.x = computerBetSpeed;
		}
		else if (computerBet.x - ball.x > 15) {
			computerBet.body.velocity.x = -computerBetSpeed;
		}
		else {
			computerBet.body.velocity.x = 0;
		}

		//Проверяем и обрабатываем столкновения мячика и ракеток
		game.physics.collide(ball, playerBet, ballHitsBet, null, this);
		game.physics.collide(ball, computerBet, ballHitsBet, null, this);

		//Проверяем, не забил ли кто-то гол
		checkGoal();
	}

	function ballHitsBet(_ball, _bet) {
		//Увеличиваем счетчик ударов шарика
		hits++;
		//С каждым третим ударом переходим на следующий уровень
		if (hits == 3) {
			nextLevel(); 
		}
		//Эффект удара
		if (ball.y < 60) {
			emitter.gravity = 5;
		} else if (ball.y > 580) {
			emitter.gravity = -5;
		}
		particleBurst(_bet.x, _bet.y);

		var diff = 0;

		if (_ball.x < _bet.x) {
			//Шарик находится с левой стороны ракетки
			diff = _bet.x - _ball.x;
			_ball.body.velocity.x = (-12 * diff);
		}
		else if (_ball.x > _bet.x) {
			//Шарик находится с правой стороны ракетки
			diff = _ball.x - _bet.x;
			_ball.body.velocity.x = (12 * diff);
		}
		else {
			//Шарик попал в центр ракетки, добавляем немножко трагической случайности его движению
			_ball.body.velocity.x = 2 + Math.random() * 8;
		}
	}

	function checkGoal() {
		if (ball.y < 15) {
			nextLevel();
			setBall();
		} else if (ball.y > 625) {
			loose();
			setBall();
		}
	}

	function setBall() {
		if (ballReleased) {
			ball.x = game.world.centerX;
			ball.y = game.world.centerY;
			ball.body.velocity.x = 0;
			ball.body.velocity.y = 0;
			ballReleased = false;
		}

	}

	function releaseBall() {
		if (!ballReleased) {
			//Увеличиваем скорость мячика с каждым ударом
			ball.body.velocity.x = ballSpeed + level * 20;
			ball.body.velocity.y = -ballSpeed - level * 20;
			ballReleased = true;
		}
	}

	function showText(txt, timeout) {
		text.setText(txt);
		setTimeout(function() {
			text.setText('');
		}, timeout);
	}

	function nextLevel() {
		level += 1;
		hits = 0;
		textLevel.setText('Level: ' + level)
		showText('Level: ' + level, 2000);
		computerBetSpeed += level * 2;
	}

	function loose() {
		level = 1;
		hits = 0;
		textLevel.setText('Level: ' + level)
		showText('You loose ', 2000);
		computerBetSpeed = 250;
	}

</script>


<!-- <h1 align="center">Hi there, I'm Michael</a> 
<img src="https://github.com/blackcater/blackcater/raw/main/images/Hi.gif" height="32"/></h1>
<h2 align="center">Programmer and game designer from Russia</h3>

<h2 align="center">My main skills:</a> 
<h2 align="center"></a> 

![Unity](https://img.shields.io/badge/unity-%23000000.svg?style=for-the-badge&logo=unity&logoColor=white)
![Unreal Engine](https://img.shields.io/badge/unrealengine-%23313131.svg?style=for-the-badge&logo=unrealengine&logoColor=white)
![C#](https://img.shields.io/badge/c%23-%23239120.svg?style=for-the-badge&logo=c-sharp&logoColor=white)

<h2 align="center">My secondary skills:</a> 
<h2 align="center"></a> 

![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=java&logoColor=white) -->

