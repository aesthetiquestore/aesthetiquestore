
{
  "name": "aesthetique-stripe-server",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "stripe": "^12.0.0"
  }
}
const express = require('express');
const cors = require('cors');
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

const app = express();
app.use(cors());
app.use(express.json());

app.post('/create-checkout-session', async (req, res) => {
  const cart = req.body.cart;
  if (!cart || cart.length === 0) {
    return res.status(400).json({ error: 'Panier vide' });
  }

  const line_items = cart.map(item => ({
    price_data: {
      currency: 'eur',
      product_data: { name: item.name },
      unit_amount: item.price * 100,
    },
    quantity: 1,
  }));

  try {
    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card'],
      mode: 'payment',
      line_items,
      success_url: 'https://aesthetiquestore.com/success',
      cancel_url: 'https://aesthetiquestore.com/cancel',
    });
    res.json({ id: session.id });
  } catch (error) {
    console.error('Erreur Stripe :', error);
    res.status(500).json({ error: 'Erreur serveur' });
  }
});

const PORT = process.env.PORT || 4242;
app.listen(PORT, () => console.log(`Serveur Stripe lancé sur le port ${PORT}`));
