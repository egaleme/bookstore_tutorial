# Build a bookstore e-commerce android app with Fuse Tools

Fuse Tools is a javascript framework for building world-class ios and android apps using UX markup and javascript.
UX markup is for navigation, layout, animation while javascript is for business and application logic. To learn more visit fusetools.com

## Getting Started
To get started you need to download Fuse Tools installer from fusetools.com and if you are using windows after Fuse Tools is installed; from the command line you run fuse install android to instal android sdk and its dependencies.

Create a new project
Once you have fuse tools installed, to create a new project you simply call 
	fuse create app ProjectName

Then run your app by calling
	fuse preview

So this point lets jump into the application and see what we are going to build out today - A Bookstore app.
In the app you can browse the list of books in the store, select one or more books to add to the cart where you can then checkout to see your purchases.

When you bootstrap a new fuse app, what you see is two files; MainView.ux and a unoproj file.
The MainView.ux file is the entry to your application while the unoproj file list your app dependecies.
When you open MianView.ux what you see 
	<App>
	</App>
What you notice is that instead of Html what you will be using is UX markup. Compared to other javascript frameworks like ReactNative and Nativescript which where built specifically for javacript developers,  fuse tools is built with collaboration of designers and developers in mind.

Now it's good practice to structure project folder into different folders depending on its functions.
	Components : will host our shared components
	Modules : our context and backend files definitions
	Pages : our app pages
	Assets : our images, fonts

Lets start by adding some data to our app. We'll be creating a mock backend for now.
## So create a Backend.js file in Modules folder
var books = [
{
	id: 1,
	title: "Beginning Android Programming",
	author: "J.F DiMarzio",
	authorbio: "About DiMarzio",
	publicationdate: "2017 by John Wiley & Sons",
	introduction: "This book is written to help start beginning Android developers ",
	picture: "Assets/books/android.png",
	cost: 25.00
},
{
	id: 2,
	title: "ES6 & Beyound",
	author: "Kyle Simpson",
	authorbio: "Kyle Simpson is a thorough pragmatist.",
	publicationdate: "2015-5-5",
	introduction: "This book is about shaking up your sense of understanding by exposing you ",
	picture: "Assets/books/es6.png",
	cost: 35.99
},
{
	id: 3,
	title: "ng-book 2",
	author: "Ari Lerner",
	authorbio: "Full stack web developer and trainer.",
	publicationdate: "2016-5-10",
	introduction: "A complete refernce book on angular 2. ",
	picture: "Assets/books/ngbook21.png",
	cost: 25.99
},
{
	id: 4,
	title: "Pro Git",
	author: "Scott Chacon and Ben Straub",
	authorbio: "Full stack web developer and trainer.",
	publicationdate: "2016-5-10",
	introduction: "Welcome to the second edition of Pro Git.  ",
	picture: "Assets/books/progit.png",
	cost: 45.99
},
{
	id: 5,
	title: "Reactjs Blueprints",
	author: "Sven A. Robbestad",
	authorbio: "Sven A. Robbestad is a developer with a keen interest in the Web .",
	publicationdate: "2016-7-10",
	introduction: "ReactJS was developed as a tool to solve a problem with the application state. ",
	picture: "Assets/books/reactjsblue.png",
	cost: 20.99
},
{
	id: 6,
	title: "ReAwaken The Giant Within",
	author: "Tony Robins",
	authorbio: "Tony Robbins is one of the great influences of this generation.",
	publicationdate: "2013-5-10",
	introduction: "I’m sending you this gift of a condensed version of my 544-page original book in the hope",
	picture: "Assets/books/awaken.png",
	cost: 22.00
},
{
	id: 7,
	title: "SurviveJS",
	author: "Juho Vapsalainen",
	authorbio: "Full stack web developer and trainer.",
	publicationdate: "2016-5-10",
	introduction: "Front-end development moves forward fast.  ",
	picture: "Assets/books/survivejs.png",
	cost: 25.99
},
{
	id: 8,
	title: "Switching To Angular2",
	author: "Minko Gechev",
	authorbio: "Minko Gechev is a software engineer who strongly believes in open source software. ",
	publicationdate: "March 2016",
	introduction: "It is the modern framework you need to build performant and robust web applications.",
				 
	picture: "Assets/books/switchingto.png",
	cost: 21.00
},
{
	id: 9,
	title: "Unlimited Sales Success",
	author: "Brian Tracy",
	authorbio: "A world class motivational and sales consultant.",
	publicationdate: "2013-2-10",
	introduction: "A complete refernce book on todays selling. ",
	picture: "Assets/books/selling.png",
	cost: 25.99
},
{
	id: 10,
	title: "Web Development with Node and ExpressJS",
	author: "Ethan Brown",
	authorbio: "A senior software engineer at PoP Art.",
	publicationdate: "2014-6-27",
	introduction: "Learn to build modern web applications with node and expressjs ",
	picture: "Assets/books/node.png",
	cost: 19.99
}
];


Lets create a promise to return the books

function getBooks() {
	return new Promise(function(resolve, reject) {
		resolve(books)
	})
}


Now lets export these functions

module.exports = {
	getBooks: getBooks
}

Now lets create a context, this will allow our pages view model to interface with it rather than directly with our backend thereby allowing object caching which will reduce unnecceary call to our backend server and so reducing bandwidth cost and saving battery.

## Lets create a Context.js file in the Modules folder

	first we create an observable to store app state

	var Backend = require("./Backend")
	var Observable = require("FuseJS/Observable")

	var store = Observable() // this will hold our data coming from the backend
	var books  = Observable() // this will hold our initial books from the store
	var cart = Observable() // this will hold the contents of our cart
	var total = Observable(0) // total price of cart items
	var isCartEmpty = Observale(true) // whether the cart is full
	var totalAmount = Observable() // total amount expression


	Lets get the books from our backend first

		function getBooks() {
			Backend.getBooks()
			.then(function(books) {
				store.replaceAll(books)
				//Now pass the books to our books observable
				store.forEach(function(book) {
					books.add(book)
					})
				})
			.catch(function(error) {
				console.log("can not load books")
				})
		}
Now call this function to load the books when the app starts
	getBooks()


Now lets create a function to add books to cart pasing the id, cost, title, picture, author and a harcoded qty of 1
	function addToCart(id, cost, title, picture, author) {
		cart.add({id: id, cost: cost, title: title, picture: picture, author: author, qty: 1})
		// Now calculate the total amount of cart items
		cart.forEach(function(book) {
			total.value = (total.value + (book.price * book.qty))
		})
		isCartEmpty.value = cart.length ? false : true
		totalAmount.value = "You Paid : $ " + total.value.toFixed(2)
	}

Now lets add a function to remove an item from the cart 

	function removeFromCart(item) {
		cart.remove(item)
		total.value = 0
		isCartEmpty.value = cart.length ? false : true
		totalAmount.value = "You Paid : $ " + total.value.toFixed(2)
	}

Now lets export these functions and observables

module.exports = {
	totalAmount,
	cart,
	isCartEmpty,
	addToCart,
	removeFromCart
}

We now have some data we can add to our app components view model
Since this app i a multi-page app, we will create a navigation component to handle routing to each of the page.

## In our MainView.ux file

<App>
<!--- responsible for routing which we'll inject to the various page -->
	<Router ux:Name="router" /> 
	<Navigator DefaultTemplate="home" >
		<Home ux:Template="home" router="router" />
		<Cart ux:Template="cart" router="router"/>
		<Detail ux:Template="detail" router="router"/>
	</Navigator>

</App>

Lets create our home page Home.ux in Pages folder and also Home.js for the buisness logic for this page

First our business and data logic for the home page

## Now we create the Home.js file in the Pages folder

var Context = require("Modules/Context");

function getBookDetail(args) {
  var book = args.data
  router.push("detail", book)
}

function goToCart() {
  router.goto("cart")
}
module.exports = {
  books: Context.books,
  getBookDetail: getBookDetail,
  goToCart: goToCart
}

## Home.ux

<Panel ux:Class="Home">
  <Router ux:Dependency="router" />
  <JavaScript File="Home.js" />
  <DockPanel>
    <DockPanel Dock="Top">
      <Panel Height="48" Color="#ea3535">
        <Text TextColor="#fff" Value="Browse Books" FontSize="20" Alignment="Center" Margin="7,5,5,5"/>
        <Image File="../Assets/cart.png" Alignment="CenterRight" Margin="5,5,7,5">
          <Tapped Handler="{goToCart}"/>
        </Image>
      </Panel>
    </DockPanel>
    <ScrollView>
      <StackPanel>
        <Each Items="{books}">
          <DockPanel>
            <Rectangle Height="1" Color="#ea353580" Opacity="0.5" Dock="Top"/>
            <DockPanel HitTestMode="LocalBounds">
              <Image File="{picture}" Dock="Left" Margin="10" Height="75" Width="75"/>
              <Grid Rows="1*,1*,1*" Margin="10">
                <Text TextColor="#000" Value="{title}" />
                <Text TextColor="#000" Value="{author}"/>
                <Text TextColor="#000" Value="{publicationdate}" />
              </Grid>
              <Tapped Handler="{getBookDetail}" />
            </DockPanel>
          </DockPanel>
        </Each>
      </StackPanel>
    </ScrollView>
  </DockPanel>
</Panel>



Lets create our detail page Detail.ux which will contain  more details about a particular book we want to buy.

First we create the buisness and data logic for the detail page.

## Detail.js

var Context = require("Modules/Context");

var book = this.Parameter; // the parameter data we passed from the home page which is an observable

var title = book.map(function(x) {
  return x.title
});

var id = book.map(function(x) {
  return x.id
});

var publicationdate = book.map(function(x) {
  return x.publicationdate
});

var author = book.map(function(x) {
  return x.author
});

var authorbio = book.map(function(x) {
  return x.authorbio
});

var picture = book.map(function(x) {
  return x.picture
});

var cost = book.map(function(x) {
  return x.cost
});

var introduction = book.map(function(x) {
  return x.introduction
});

var price = book.map(function(x) {
  return "Price : $" +x.cost
});

function addToCart() {
  Context.addToCart(id.value, cost.value, title.value, picture.value, author.value);
  router.goBack()
}

module.exports = {
  title,
  id,
  introduction,
  picture,
  cost,
  authorbio,
  publicationdate,
  author,
  addToCart,
  price
}

## Detail.ux

<Panel ux:Class="Detail" >
  <Router ux:Dependency="router" />
  <JavaScript File="Detail.js" />
  <DockPanel>
  <DockPanel Dock="Top">
    <Panel Height="48" Color="#ea3535">
      <Text TextColor="#fff" Value="{title}" Alignment="Center" Margin="7,5,5,5"/>
      <Image File="../Assets/cart.png" Alignment="CenterRight" Margin="5,5,7,5" />
    </Panel>
  </DockPanel>
  <ScrollView>
    <StackPanel>
      <DockPanel>
        <Grid RowCount="3" Margin="10" Alignment="Center">
          <BlackText Value="{title}"/>
          <BlackText Value="{publicationdate}"/>
          <BlackText Value="{author}"/>
        </Grid>
      </DockPanel>
      <StackPanel Margin="10">
        <Rectangle Height="1" Color="#ea353580" Opacity="0.5" Margin="10"/>
        <Image File="{picture}" Margin="10" Alignment="Center" />
        <BlackText Value="{authorbio}" Alignment="Center"/>
        <TextView TextWrapping="Wrap" Value="{introduction}" Margin="10" TextColor="#000" Alignment="Center" />
        <BlackText Value="{price}" Alignment="Center" Margin="20" />
        <Rectangle HitTestMode="LocalBounds" Color="#ea3535" Height="40" Width="100%" Margin="10" CornerRadius="10">
          <Text TextColor="#fff" Value="Add To Cart" Alignment="Center" />
          <Tapped Handler="{addToCart}"/>
        </Rectangle>
      </StackPanel>
    </StackPanel>
  </ScrollView>
  </DockPanel>
</Panel>

Now we create our cart page along with the cart business and data logic.

First the Cart.js file in the Pages folder for the cart page business logic

var Context = require("Modules/Context");

function remove(args) {
  var item = args.data
  Context.removeFromCart(item)
}

function goHome() {
  router.goto("home")
}

var todaysDate = "Purchases made on " + new Date().toDateString() + " at " + new Date().toLocaleTimeString();

var qty = Context.cart.count() // this will return the number of items in cart as an observable 

module.exports = {
  goHome: goHome,
  noItems: Context.isCartEmpty,
  cart: Context.cart,
  remove: remove,
  todaysDate: todaysDate,
  totalAmount: Context.totalAmount,
  qty: qty
}


## Cart.ux

<Panel ux:Class="Cart">
  <Router ux:Dependency="router" />
  <JavaScript File="Cart.js" />
  <DockPanel>
    <DockPanel Dock="Top">
      <Panel Height="48" Color="#ea3535">
        <Text TextColor="#fff"  Value="Home" Alignment="CenterLeft" Margin="7,5,5,5">
          <Tapped Handler="{goHome}"/>
        </Text>
        <Text TextColor="#fff" Value="Your cart" Alignment="Center" Margin="7,5,5,5"/>
        <Image File="../Assets/cart.png" Alignment="CenterRight" Margin="5,5,7,5" />
      </Panel>
    </DockPanel>
    <DockPanel>
      <Panel Dock="Top" Margin="10">
        <Panel ux:Name="noBooks" Color="#000" Opacity="0.4">
          <Text TextColor="#fff" Alignment="Center" Value="No books in the cart" />
          <!--- If there are content in the cart don't show this component -->
          <WhileFalse Value="{noItems}">
            <Scale Factor="0"/>
            <Change noBooks.Opacity="0"/>
          </WhileFalse>
        </Panel>
        <Panel ux:Name="checkout" Color="#000" Opacity="0.4">
          <StackPanel>
            <Text TextColor="#fff" Value="{todaysDate}" Alignment="Center" Margin="10"/>
            <StackPanel Orientation="Horizontal" Alignment="Center">
              <Text TextColor="White" Value="Quantity of items :" Alignment="Center" Margin="10" />
              <Text TextColor="#fff" Value="{qty}" Alignment="Center" Margin="10"/>
            </StackPanel>
            <Text TextColor="#fff" Value="{totalAmount}" Alignment="Center" Margin="10" />
          </StackPanel>
          <!--- If there are no content in the cart don't show this component -->
          <WhileTrue Value="{noItems}">
            <Scale Factor="0" />
            <Change checkout.Opacity="0" />
          </WhileTrue>
        </Panel>
      </Panel>
      <ScrollView>
        <StackPanel>
          <Each Items="{cart}">
            <DockPanel>
              <Rectangle Height="1" Color="#ea353580" Opacity="0.3" Dock="Top"/>
              <Image File="{picture}" Dock="Left" Height="75" Width="75" Margin="10"/>
              <Grid Rows="1*,1*,1*" Margin="10">
                <BlackText Value="{title}" />
                <BlackText Value="{author}"/>
                <StackPanel Orientation="Horizontal">
                  <BlackText Value="$"/>
                  <BlackText Value="{cost}" />
                  <Rectangle  HitTestMode="LocalBounds" Margin="10,0,0,0" Alignment="Right" ux:Name="removeBtn" Color="#ea3535" Height="15">
                    <Text TextColor="White" FontSize="10" Value="Remove" Alignment="Center" />
                    <Tapped Handler="{remove}" />
                  </Rectangle>
                </StackPanel>
              </Grid>
            </DockPanel>
          </Each>
        </StackPanel>
      </ScrollView>
    </DockPanel>
  </DockPanel>
</Panel>

To build our app on our android device for testing; make sure you have android adb usb driver installed
Then connect your device to the pc and use the command
Fuse build bookstore-tutorial.unoporj -tandroid -DGRADLE -r

Please checkout the tutorial app on github.com/egaleme/bookstore-tutorial

Talk to Me
And that’s it, a scalable and performant e-commerce app that you can easily add to and enhance for your needs. If you have any questions or comments then please let me know.