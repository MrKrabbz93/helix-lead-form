const nodemailer = require('nodemailer');

exports.handler = async (event, context) => {
  // Only allow POST requests
  if (event.httpMethod !== 'POST') {
    return { statusCode: 405, body: 'Method Not Allowed' };
  }

  try {
    const data = JSON.parse(event.body);

    // --- 1. HONEYPOT CHECK (Security) ---
    // If the hidden field '_gotcha' has any value, it's a bot.
    // We return success to fool the bot, but do not process the data.
    if (data._gotcha) {
      console.warn(`[Spam Detected] Honeypot triggered by IP: ${event.headers['client-ip']}`);
      return {
        statusCode: 200,
        body: JSON.stringify({ message: 'Submission received.' }),
      };
    }

    // --- 2. VALIDATION (Basic) ---
    if (!data.email || !data.email.includes('@')) {
      return { statusCode: 400, body: JSON.stringify({ error: 'Invalid email address.' }) };
    }
    if (!data.fullName) {
      return { statusCode: 400, body: JSON.stringify({ error: 'Name is required.' }) };
    }

    // --- 3. STORAGE (Simulation/Placeholder) ---
    // In a full production app, you would use Airtable, Firebase, or Google Sheets API here.
    // For this deployment, we log the secure capture to the server logs.
    console.log('[Helix Secure Storage] Persisting lead:', {
      timestamp: new Date().toISOString(),
      client: data.brandName,
      lead: data.email,
      budget: data.budget
    });

    // --- 4. NOTIFICATION (Email) ---
    // SECURITY NOTE: In this demo/builder configuration, we accept the 'notificationEmail' 
    // from the client-side config to allow for rapid customization without redeployment.
    // 
    // [PRODUCTION WARNING]: In a high-security production environment, the destination email 
    // MUST be stored in the Serverless Environment Variables (e.g., process.env.CLIENT_EMAIL) 
    // rather than passed from the client, to prevent Open Relay abuse by malicious actors.
    
    if (process.env.SMTP_HOST && data.notificationEmail) {
      const transporter = nodemailer.createTransport({
        host: process.env.SMTP_HOST,
        port: process.env.SMTP_PORT || 587,
        secure: false, // true for 465, false for other ports
        auth: {
          user: process.env.SMTP_USER,
          pass: process.env.SMTP_PASS,
        },
      });

      // Construct the email body
      const emailHtml = `
        <h2>New High-Ticket Lead: ${data.fullName}</h2>
        <p><strong>Source:</strong> ${data.brandName} Rapid Impact Form</p>
        <hr />
        <h3>Contact Details</h3>
        <ul>
          <li><strong>Email:</strong> ${data.email}</li>
          <li><strong>Phone:</strong> ${data.phone || 'N/A'}</li>
        </ul>
        <h3>Qualification Data</h3>
        <ul>
          <li><strong>Budget:</strong> ${data.budget || 'N/A'}</li>
          <li><strong>Timeline:</strong> ${data.timeline || 'N/A'}</li>
          <li><strong>Brief:</strong> ${data.description || 'N/A'}</li>
        </ul>
      `;

      await transporter.sendMail({
        from: '"Helix Engine" <notifications@yourdomain.com>',
        to: data.notificationEmail,
        subject: `New Lead: ${data.fullName} (${data.budget || 'Inquiry'})`,
        html: emailHtml,
      });
      console.log(`[Notification] Email sent to ${data.notificationEmail}`);
    } else {
      console.log('[Notification] Skipped: No SMTP config or Notification Email provided.');
    }

    // --- 5. RESPONSE ---
    return {
      statusCode: 200,
      body: JSON.stringify({ status: 'success', message: 'Lead securely processed by Helix.' }),
    };

  } catch (error) {
    console.error('[Error] Processing failed:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal Server Error' }),
    };
  }
};
