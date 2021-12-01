# hoyYeahiDid
phpcheckout
<?php
session_start();
//Error reporting
error_reporting(E_ALL);
ini_set('display_errors','1');
include "storescripts/mysql.php"; 
?>
<?php 
///////////////////////////////////////////////////////////////////////
//       Section 1  (if user attempts to add something to the cart)
///////////////////////////////////////////////////////////////////////
if (isset($_POST['pid'])) {
    $pid = $_POST['pid'];
	$wasFound = false;
	$i = 0;
	// If the cart session variable is not set or cart array is empty
	if (!isset($_SESSION["cart_array"]) || count($_SESSION["cart_array"]) < 1) { 
	    // RUN IF THE CART IS EMPTY OR NOT SET
		$_SESSION["cart_array"] = array(0 => array("item_id" => $pid, "quantity" => 1));
	} else {
		// RUN IF THE CART HAS AT LEAST ONE ITEM IN IT
		foreach ($_SESSION["cart_array"] as $each_item) { 
		      $i++;
		      while (list($key, $value) = each($each_item)) {
				  if ($key == "item_id" && $value == $pid) {
					  // That item is in cart already so let's adjust its quantity using array_splice()
					  array_splice($_SESSION["cart_array"], $i-1, 1, array(array("item_id" => $pid, "quantity" => $each_item['quantity'] + 1)));
					  $wasFound = true;
				  } // close if condition
		      } // close while loop
	       } // close foreach loop
		   if ($wasFound == false) {
			   array_push($_SESSION["cart_array"], array("item_id" => $pid, "quantity" => 1));
		   }
	}
	header("location: cart.php"); 
    exit();
}
?>
<?php 
///////////////////////////////////////////////////////////////////////
//       Section 2  (if user chooses to empty their shopping cart)
///////////////////////////////////////////////////////////////////////
if (isset($_GET['cmd']) && $_GET['cmd'] == "emptycart") {
    unset($_SESSION["cart_array"]);
}
?>
<?php 
///////////////////////////////////////////////////////////////////////
//       Section 3  (if user chooses to adjust item quantity)
///////////////////////////////////////////////////////////////////////
if (isset($_POST['book_to_adjust']) && $_POST['book_to_adjust'] != "") {

	
	$itemToAdjust = $_POST['book_to_adjust'];
	$quantity = $_POST['quantity'];
	$quantity = preg_replace('#[^0-9]#i', '', $quantity);
	if ($quantity >=100){$quantity = 99;}
	if ($quantity <1){$quantity = 1;}
	$i = 0;
    foreach ($_SESSION["cart_array"] as $each_item) { 
		      $i++;
		      while (list($key, $value) = each($each_item)) {
				  if ($key == "item_id" && $value == $itemToAdjust) {
					  // That item is in cart already so let's adjust its quantity using array_splice()
					  array_splice($_SESSION["cart_array"], $i-1, 1, array(array("item_id" => $itemToAdjust, "quantity" => $quantity)));
				  } // close if condition
		      } // close while loop
	       } // close foreach loop
		   header("location: cart.php"); 
    exit();
}
?>
<?php
/////////////////////////////////////////////////////////////////////////
//       Section 4 (if user wants to remove item from cart
/////////////////////////////////////////////////////////////////////////
if (isset($_POST['book_to_remove']) && $_POST['book_to_remove']!=""){
	
	$key_to_remove = $_POST['book_to_remove'];
	if (count($_SESSION["cart_array"])<=1){
		unset($_SESSION["cart_array"]);
	}else{
		unset($_SESSION["cart_array"]["$key_to_remove"]);
		sort($_SESSION["cart_array"]);
	
	
}
}
?>
<?php
/////////////////////////////////////////////////////////////////////////
//       Section 5 (render the cart for the user to view on the page)
/////////////////////////////////////////////////////////////////////////
$cartOutput = "";
$cartTotal = "";
if (!isset($_SESSION["cart_array"]) || count($_SESSION["cart_array"]) < 1) {
    $cartOutput = "<h2 align='center'>Your shopping cart is empty</h2>";
} else {
	$i = 0; 
    foreach ($_SESSION["cart_array"] as $each_item) { 
	
	$item_id= $each_item['item_id'];
	$sql = mysql_query("SELECT * FROM books WHERE bookId='$item_id' LIMIT 1");
	while ($row = mysql_fetch_array($sql)){
		$bookName = $row['bookName'];
		$price = $row['price'];
		$formatId =$row['formatId'];
		$bookId =$row['bookId'];
		
		
		
	}
	
$resul3 = mysql_query("SELECT format.formatType FROM format JOIN books ON books.formatId = format.formatId WHERE format.formatId = $formatId");
  				while ($row = mysql_fetch_array($resul3)) {	
				$formatType = $row["formatType"];	
				}
				$resul4 = mysql_query("SELECT books.*, stock.* FROM books JOIN stock ON books.bookId = stock.bookId WHERE stock.bookId = $bookId");
  				while ($row = mysql_fetch_array($resul4)) {	
				$stockLevel = $row["stockLevel"];
				}
				
	$pricetotal = $price * $each_item['quantity'];
	$cartTotal = $pricetotal + $cartTotal;
	
	setlocale(LC_MONETARY, 'en_GB');
	$pricetotal = money_format("%10.2n",$pricetotal);
	// Dynamic Table Assembly
	$cartOutput .= '<tr>';
	$cartOutput .= "<td><a href=\"book_product.php?bookId=$item_id\">" .$bookName.'<br/><img src="inventory_images/Books/' . $bookId . '.jpg" alt="'.$bookName.'" width="40" height"40" border"1"/></td>';
	$cartOutput .= '<td><form action="cart.php" method = "post">
	<input name="quantity" " type="text" value="' .$each_item['quantity']. '" size="1" maxlength="2"/>
	<input name="adjustBtn'. $item_id . '" type="submit" value="Update"/>
	<input name="book_to_adjust" type="hidden" value = "'. $item_id . '"/>
	</form></td>';
	//$cartOutput .= '<td>' .$each_item['quantity']. '</td>';
	
	$cartOutput .= '<td align="center">£' .$price. '</td>';
	$cartOutput .= '<td align="center">' .$formatType. '</td>';
	$cartOutput .= '<td align="center">' .$pricetotal. '</td>';
	$cartOutput .= '<td align="center"><form action="cart.php" method = "post"><input name="removeBtn'. $item_id . '" type="submit" value="X"/><input name="book_to_remove" type="hidden" value = "'.$i.'"/></form></td>';
	$cartOutput .= '</tr>';
	$i++;

	}
	setlocale(LC_MONETARY, 'en_GB');
	$cartTotal = money_format("%10.2n",$cartTotal);
	$cartTotal = "<div align='right'>Cart Total :  ".$cartTotal. "</div>";
}
?>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<title>Shopping Cart</title>
<link href="styles/abcMain.css" rel="stylesheet" type="text/css" />
</head>

<body>
<div id="container">
	<div id="header">
   
    <img src="Images/logo.png" width="200" height="200" /></div>
    
	 <?php include_once("NavTemplates/NavTemp.php");?>
			<div id="content">
	
   <h2>
					Your Shopping Cart
				</h2>
    
    <br /><br />
    <table width="100%"  border="1" cellspacing="0" cellpadding="6">
      <tr>
        <td bgcolor="#CCCCCC" width="38%" height="40" align="center"><strong>Book Name</strong></td>
        <td bgcolor="#CCCCCC" align="center" width="11%"><strong>Quantity</strong></td>
        <td bgcolor="#CCCCCC" align="center" width="6%"><strong>Price</strong></td>
        <td bgcolor="#CCCCCC" align="center" width="12%"><strong>Format</strong></td>
        <td bgcolor="#CCCCCC"  align="center" width="11%"><strong>SubTotal</strong></td>
         <td bgcolor="#CCCCCC" align="center" width="6%"><strong>Remove</strong></td>
      </tr>
      
      <?php echo $cartOutput ?>
      <!--<tr>
        <td>&nbsp;</td>
        <td>&nbsp;</td>
        <td>&nbsp;</td>
        <td>&nbsp;</td>
        <td>&nbsp;</td>
        
      </tr> -->
    </table>
    <br/>
    <?php echo $cartTotal ?>
    <br /><br /> <br /><br /><br /><br />
    <a  href="cart.php?cmd=emptycart"><img align="absmiddle"src="Images/Empty_cart.png" width="130" height="50" alt="cart" /></a>
			</div>
            
			<div id="aside">
	
			</div>
			<div id="footer">
				Copyright © ABC Books |<a href="_admin/index.php">Admin</a>
			</div>
		</div>
  </div>
</div>




</body>
</html>
