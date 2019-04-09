<template>
  <div>
    <v-container grid-list-lg>
      <v-layout row wrap>
        <v-flex xs12 sm6 md3>
          <v-btn large color="info" @click="createPDF">PDF</v-btn>
        </v-flex>
      </v-layout>
      <v-layout row wrap>
        <v-flex>
          <code class="pa-3">{{ order }}</code>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>
  
<script>
import { order } from "../order.js";
import { lines } from "../lines.js";
import * as jsPDF from "jspdf";
import "jspdf-autotable";
export default {
  data: () => ({
    msg: "Message",
    order,
    lines,
    myNestedArrays: []
  }),
  methods: {
    createPDF() {
      //       // circle for positioning
      // doc.setFillColor('#ff0000')
      // doc.circle(sc(y),sc(y),3, 'F')

      const docConfig = {
        orientation: "p",
        unit: "px",
        format: "leter"
      };
      let doc = new jsPDF(docConfig);
      //scale function
      function sc(val) {
        // (72px/96px)
        return val * 0.75;
      }

      function divider(y) {
        // y = 170;
        return doc.line(
          sc(layout.leftMargin),
          sc(y),
          sc(layout.rightMargin - layout.leftMargin),
          sc(y)
        );
      }

      const layout = {
        fontSize: 16,
        width: 612,
        height: 796,
        leftMargin: 18,
        rightMargin: 594,
        col2: 117,
        col3: 216,
        col4: 315,
        col5: 414,
        col6: 513
      };

      const m = sc(layout.leftMargin); // default margin
      const col2 = sc(layout.col2);
      const col3 = sc(layout.col3);
      const col4 = sc(layout.col4);
      const col5 = sc(layout.col5);
      const col6 = sc(layout.col6);
      const w = sc(layout.width);
      const h = sc(layout.height);
      const fs = layout.fontSize; // default font size in px
      const sfs = sc(layout.fontSize); // scaled default font size
      const tc = { baseline: "top" }; // text config cont
      const o = this.order;

      doc.setFont("times");

      // Header
      // Filled red square
      doc.setDrawColor(0);
      doc.setFillColor("#005595");
      doc.rect(0, 0, w, sc(120), "F");

      doc.setFontSize(sc(24));
      doc.setTextColor("#ffffff");
      let y = 27;
      doc.text(`DascoLaser.com`, m, sc(y), tc);
      doc.text(`Order ${o.order_no}`, m, sc(y + 24), tc);

      doc.setFontSize(sc(12));
      // Daughtridge Sales Info
      doc.text(`Daughtridge Sales`, col3, sc(y + 6), tc);
      doc.text(`laser@dascosales.com`, col3, sc(y + 6 + fs * 1), tc);
      doc.text(`600 S Grace St`, col3, sc(y + 6 + fs * 2), tc);
      doc.text(`Rocky Mount, NC 27803`, col3, sc(y + 6 + fs * 3), tc);
      doc.text(`(252) 977-1131`, col3, sc(y + 6 + fs * 4), tc);

      doc.text(`Date Ordered: ${o.created_at}`, col5, sc(y + 6 + fs * 1), tc);
      doc.text(`Order Status: ${o.order_status}`, col5, sc(y + 6 + fs * 2), tc);
      doc.text(
        `Payment Status: ${o.order_payment_status}`,
        col5,
        sc(y + 6 + fs * 3),
        tc
      );

      // divider(120)

      // Buyer
      doc.setFontSize(sfs);
      doc.setTextColor("black");
      doc.setFontStyle("bold");
      y = 130; // top y
      doc.text("Customer:", m, sc(y), tc);
      doc.setFontStyle("normal");
      doc.text(o.user_name, m, sc(y + fs * 1), tc);
      doc.text(o.email, m, sc(y + fs * 2), tc);
      doc.text(o.phone, m, sc(y + fs * 3), tc);
      doc.text(`Cust No: nav_customer_no?`, m, sc(y + fs * 4), tc);

      // Shipping
      doc.setFontSize(sfs);
      doc.setFontStyle("bold");
      doc.text("Shipping:", col3, sc(y), tc);
      doc.text(o.shipping_info.company, col3, sc(y + fs * 1), tc);
      doc.setFontStyle("normal");
      doc.text(o.shipping_info.address1, col3, sc(y + fs * 2), tc);
      const sCityStateZip = `${o.shipping_info.city}, ${
        o.shipping_info.state
      } ${o.shipping_info.zipcode}`;
      if (o.shipping_info.address2 == "") {
        doc.text(sCityStateZip, col3, sc(y + fs * 3), tc);
      } else {
        doc.text(o.shipping_info.address2, col3, sc(y + fs * 3), tc);
        doc.text(sCityStateZip, col3, sc(y + fs * 4), tc);
      }

      // Billing
      doc.setFontSize(sfs);
      doc.setFontStyle("bold");
      doc.text("Billing:", col5, sc(y), tc);
      doc.text(o.billing_info.company, col5, sc(y + fs * 1), tc);
      doc.setFontStyle("normal");
      doc.text(o.billing_info.address1, col5, sc(y + fs * 2), tc);
      const bCityStateZip = `${o.billing_info.city}, ${o.billing_info.state} ${
        o.billing_info.zipcode
      }`;
      if (o.billing_info.address2 == "") {
        doc.text(bCityStateZip, col5, sc(y + fs * 3), tc);
      } else {
        doc.text(o.billing_info.address2, col5, sc(y + fs * 3), tc);
        doc.text(bCityStateZip, col5, sc(y + fs * 4), tc);
      }

      divider(220);

      y = 230;
      // Items Table
      doc.autoTable({
        margin: { top: sc(y) },
        theme: "plain",
        columnStyles: {
          quantity: { halign: "center" },
          price: { halign: "right" },
          total: { halign: "right" }
        },
        head: [
          {
            item_no: "Item",
            model: "Model",
            price: "Price",
            quantity: "Qty",
            total: "Total"
          }
        ],
        body: this.arrayOfItems(o.items)
      });

      y =
        doc.autoTable.previous.finalY +
        doc.autoTable.previous.headHeight +
        doc.autoTable.previous.height +
        20;
      divider(y);

      // Totals
      doc.autoTable({
        startY: sc(y + 20),
        margin: { left: col5 },
        theme: "plain",
        columnStyles: { amount: { halign: "right" } },
        // head: [{ item_no:'Item' , model: 'Model', price: 'Price' , quantity: 'Qty', total: 'Total'}],
        body: [
          {
            name: "Subtotal",
            amount: o.subtotal_price.toFixed(2)
          },
          {
            name: "Tax",
            amount: o.total_tax.toFixed(2)
          },
          {
            name: "Shipping",
            amount: o.billed_shipping_cost.toFixed(2)
          },
          {
            name: "Total",
            amount: o.total_price.toFixed(2)
          }
        ]
      });
      console.log(doc.autoTable.previous);

      doc.output("dataurlnewwindow");

      // doc.save(`Invoice-${orderNo}.pdf`)
    },
    arrayOfItems(val) {
      let nested = [];
      val.forEach(i => {
        let obj = {
          item_no: i.item_no,
          model: i.model,
          price: i.price.toFixed(2),
          quantity: i.quantity,
          total: (i.price * i.quantity).toFixed(2)
        };
        nested.push(obj);
      });

      return nested;
    }
  }
};
</script>
