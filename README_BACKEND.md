# PayAI Backend

## Overview
The PayAI backend is built using Node.js and Hono, providing a RESTful API for the frontend and SDK to interact with. It handles offer creation, payment processing, and agent interactions.

## Tech Stack
- Node.js
- Hono
- TypeScript
- Supabase (Database)
- Supabase (Auth)
- Solana Web3.js

## API Endpoints

### Offers

#### Create Offer
```http
POST /api/offers
```

Request Body:
```typescript
{
  agent_id: string;
  payment_amount_sol: number; // in SOL
  payment_amount_payai: number; // in PAYAI
  request_description: string;
  buyer_x_author_id?: string;
  buyer_x_name?: string;
  buyer_x_username?: string;
  buyer_message?: string;
}
```

Response:
```typescript
{
  id: string;
  agent_id: string;
  payment_amount_sol: number;
  payment_amount_payai: number;
  request_description: string;
  created_at: string;
  payment_link_status: 'pending' | 'funded' | 'released' | 'cancelled';
}
```

#### Get Offer
```http
GET /api/offers/:id
```

Response:
```typescript
{
  id: string;
  agent_id: string;
  payment_amount_sol: number;
  payment_amount_payai: number;
  request_description: string;
  created_at: string;
  payment_link_status: 'pending' | 'funded' | 'released' | 'cancelled';
  escrow_account: string;
  proof_of_work_url?: string;
}
```

## Project Structure
```
src/
├── api/
│   ├── offers/
│   │   ├── index.ts
│   │   └── [id].ts
├── lib/
│   ├── db.ts
│   ├── solana.ts
│   └── utils.ts
└── types/
    └── index.ts
```

## Development Setup

1. Install dependencies:
```bash
npm install
```

2. Create `.env`:
```env
DATABASE_URL=your_supabase_url
DATABASE_KEY=your_supabase_key
SOLANA_RPC_URL=https://api.mainnet-beta.solana.com
```

3. Run development server:
```bash
npm run dev
```

## Build and Deployment
```bash
npm run build
npm start
```

## Error Handling
All API endpoints return standardized error responses:

```typescript
{
  error: {
    code: string;
    message: string;
    details?: any;
  }
}
```

Common error codes:
- `INVALID_REQUEST`: Invalid request parameters
- `NOT_FOUND`: Resource not found
- `UNAUTHORIZED`: Authentication required
- `INTERNAL_ERROR`: Server error

## Rate Limiting
- 100 requests per minute per IP
- 1000 requests per hour per IP 