# bill-mob

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mobile Shop Billing</title>

<style>
body {
    font-family: Arial, sans-serif;
    background: #eef3f8;
    margin: 0;
    padding: 20px;
    color: #243746;
}
.container {
    max-width: 950px;
    margin: auto;
    background: white;
    padding: 22px;
    border-radius: 12px;
    box-shadow: 0 3px 12px #ccd5df;
}
h1, h2 {
    text-align: center;
    color: #123b59;
}
input, select {
    width: 100%;
    padding: 10px;
    margin: 6px 0 12px;
    box-sizing: border-box;
    border: 1px solid #ccd5df;
    border-radius: 5px;
}
button {
    padding: 10px 15px;
    border: none;
    border-radius: 5px;
    background: #087f8c;
    color: white;
    cursor: pointer;
    margin: 4px;
}
.remove { background: #c0392b; }
table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 15px;
}
th, td {
    border: 1px solid #d5dde5;
    padding: 8px;
    text-align: left;
    font-size: 13px;
}
th { background: #e1f1f4; }
.total {
    text-align: right;
    line-height: 1.8;
    margin-top: 18px;
}
@media(max-width:600px) {
    body { padding: 7px; }
    .container { padding: 12px; }
    table { font-size: 11px; }
    th, td { padding: 5px; }
}
@media print {
    body { background: white; padding: 0; }
    .container { box-shadow: none; }
    .no-print { display: none !important; }
}
</style>
</head>

<body>
<div class="container">

<h1>Mobile Shop Billing System</h1>

<div class="no-print">
<h2>Shop Details</h2>

<input id="shop" placeholder="Shop Name">
<input id="address" placeholder="Shop Address">
<input id="shopPhone" placeholder="Shop Phone Number">

<h2>Customer Details</h2>
<input id="customer" placeholder="Customer Name">
<input id="customerPhone" placeholder="Customer Phone">

<h2>Add Product</h2>

<label>Category</label>
<select id="category" onchange="toggleIMEI()">
    <option value="Phone">Mobile Phone</option>
    <option value="Earbuds">Earbuds</option>
    <option value="Accessory">Accessories</option>
</select>

<input id="brand" placeholder="Brand Name">
<input id="model" placeholder="Product / Model Name">

<div id="imeiBox">
    <input id="imei" placeholder="Phone IMEI Number">
</div>

<input id="price" type="number" min="0"
       step="0.01" placeholder="Price (₹)">
<input id="qty" type="number" min="1"
       step="1" value="1" placeholder="Quantity">
<input id="discount" type="number" min="0"
       max="100" value="0" placeholder="Discount (%)">
<input id="tax" type="number" min="0"
       max="100" value="0" placeholder="Tax / GST (%)">

<button onclick="addProduct()">Add Product</button>
<button onclick="clearProducts()">Clear Products</button>
</div>

<hr>

<h2>Customer Bill</h2>
<p><b>Bill No:</b> <span id="billNo"></span></p>
<p><b>Date:</b> <span id="date"></span></p>

<p><b>Shop:</b> <span id="showShop"></span></p>
<p><b>Address:</b> <span id="showAddress"></span></p>
<p><b>Shop Phone:</b> <span id="showShopPhone"></span></p>
<p><b>Customer:</b> <span id="showCustomer"></span></p>
<p><b>Customer Phone:</b> <span id="showCustomerPhone"></span></p>

<table>
<thead>
<tr>
    <th>Product</th>
    <th>Brand</th>
    <th>IMEI</th>
    <th>Qty</th>
    <th>Price</th>
    <th>Amount</th>
    <th class="no-print">Remove</th>
</tr>
</thead>
<tbody id="billItems">
<tr><td colspan="7">No products added</td></tr>
</tbody>
</table>

<div class="total">
    <p>Subtotal: ₹<span id="subtotal">0.00</span></p>
    <p>Discount: ₹<span id="discountTotal">0.00</span></p>
    <p>Tax / GST: ₹<span id="taxTotal">0.00</span></p>

    <div class="no-print">
        <label>Other Charges (₹)</label>
        <input id="charges" type="number" min="0"
               value="0" oninput="renderBill()">
        <label>Payment Method</label>
        <select id="payment">
            <option>Cash</option>
            <option>UPI</option>
            <option>Card</option>
        </select>
    </div>

    <p class="print-only" id="printPayment"></p>
    <h2>Total: ₹<span id="total">0.00</span></h2>
</div>

<div class="no-print">
    <button onclick="downloadPDF()">Download / Print PDF</button>
    <button onclick="newBill()">New Bill</button>
</div>

<p style="text-align:center">Thank you for shopping with us!</p>

</div>

<script>
let products = [];
let nextBill = 1;

document.getElementById("billNo").textContent =
    "MS-" + Date.now().toString().slice(-8);

document.getElementById("date").textContent =
    new Date().toLocaleDateString("en-IN");

function toggleIMEI() {
    const phone =
        document.getElementById("category").value === "Phone";

    document.getElementById("imeiBox").style.display =
        phone ? "block" : "none";

    if (!phone) {
        document.getElementById("imei").value = "";
    }
}

function addProduct() {
    const category = document.getElementById("category").value;
    const brand = document.getElementById("brand").value.trim();
    const model = document.getElementById("model").value.trim();
    const imei = document.getElementById("imei").value.trim();
    const priceText = document.getElementById("price").value;
    const price = Number(priceText);
    const qty = Number(document.getElementById("qty").value);
    const discount = Number(document.getElementById("discount").value);
    const tax = Number(document.getElementById("tax").value);

    if (!brand || !model || priceText === "") {
        alert("Enter brand, model and price.");
        return;
    }

    if (!Number.isFinite(price) || price < 0 ||
        !Number.isInteger(qty) || qty < 1) {
        alert("Enter a valid price and quantity.");
        return;
    }

    if (discount < 0 || discount > 100 ||
        tax < 0 || tax > 100) {
        alert("Discount and tax must be between 0 and 100.");
        return;
    }

    if (category === "Phone" && !imei) {
        alert("Please enter the phone IMEI number.");
        return;
    }

    products.push({
        category, brand, model, imei, price, qty, discount, tax
    });

    renderBill();

    ["brand", "model", "imei", "price"].forEach(id => {
        document.getElementById(id).value = "";
    });

    document.getElementById("qty").value = 1;
    document.getElementById("discount").value = 0;
    document.getElementById("tax").value = 0;
}

function renderBill() {
    const body = document.getElementById("billItems");
    body.innerHTML = "";

    let subtotal = 0;
    let discountTotal = 0;
    let taxTotal = 0;

    products.forEach((p, i) => {
        const gross = p.price * p.qty;
        const discountAmount = gross * p.discount / 100;
        const taxable = gross - discountAmount;
        const taxAmount = taxable * p.tax / 100;
        const amount = taxable + taxAmount;

        subtotal += gross;
        discountTotal += discountAmount;
        taxTotal += taxAmount;

        const row = document.createElement("tr");
        [
            p.model + " (" + p.category + ")",
            p.brand,
            p.imei || "-",
            p.qty,
            money(p.price),
            money(amount)
        ].forEach(value => {
            const cell = document.createElement("td");
            cell.textContent = value;
            row.appendChild(cell);
        });

        const action = document.createElement("td");
        action.className = "no-print";
        const button = document.createElement("button");
        button.textContent = "Remove";
        button.className = "remove";
        button.onclick = () => {
            products.splice(i, 1);
            renderBill();
        };
        action.appendChild(button);
        row.appendChild(action);
        body.appendChild(row);
    });

    if (products.length === 0) {
        body.innerHTML =
            '<tr><td colspan="7">No products added</td></tr>';
    }

    const charges = Math.max(
        0, Number(document.getElementById("charges").value) || 0
    );

    setText("subtotal", subtotal.toFixed(2));
    setText("discountTotal", discountTotal.toFixed(2));
    setText("taxTotal", taxTotal.toFixed(2));
    setText("total",
        (subtotal - discountTotal + taxTotal + charges).toFixed(2));

    syncDetails();
}

function money(value) {
    return "₹" + Number(value).toFixed(2);
}

function setText(id, value) {
    document.getElementById(id).textContent = value;
}

function syncDetails() {
    const fields = {
        shop: "showShop",
        address: "showAddress",
        shopPhone: "showShopPhone",
        customer: "showCustomer",
        customerPhone: "showCustomerPhone"
    };

    for (const [inputId, outputId] of Object.entries(fields)) {
        setText(outputId,
            document.getElementById(inputId).value || "-");
    }

    setText("printPayment",
        "Payment Method: " + document.getElementById("payment").value);
}

["shop", "address", "shopPhone", "customer",
 "customerPhone", "payment"].forEach(id => {
    document.getElementById(id).addEventListener("input", syncDetails);
    document.getElementById(id).addEventListener("change", syncDetails);
});

function clearProducts() {
    if (!confirm("Remove all products from this bill?")) return;
    products = [];
    renderBill();
}

function downloadPDF() {
    if (products.length === 0) {
        alert("Add at least one product before printing.");
        return;
    }
    syncDetails();
    window.print();
}

function newBill() {
    if (!confirm("Start a new bill? Current products will be cleared.")) {
        return;
    }

    products = [];
    nextBill++;

    setText("billNo",
        "MS-" + Date.now().toString().slice(-8));
    setText("date", new Date().toLocaleDateString("en-IN"));

    ["customer", "customerPhone"].forEach(id => {
        document.getElementById(id).value = "";
    });

    document.getElementById("charges").value = 0;
    renderBill();
}

syncDetails();
toggleIMEI();
renderBill();
</script>
</body>
</html>
