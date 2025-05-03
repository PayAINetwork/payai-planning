# PayAI Database

## Overview
The PayAI database is built on Supabase using PostgreSQL. It stores all the necessary information for offers and agents.

## Database Schema

### Offers Table
```sql
CREATE TABLE offers (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    agent_id TEXT NOT NULL,
    payment_amount_sol DECIMAL NOT NULL,
    payment_amount_payai DECIMAL NOT NULL,
    request_description TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    payment_link_status TEXT NOT NULL CHECK (
        payment_link_status IN (
            'pending',
            'funded',
            'work_started',
            'work_delivered',
            'payment_complete'
        )
    ),
    proof_of_work_url TEXT,
    buyer_x_author_id TEXT,
    buyer_x_name TEXT,
    buyer_x_username TEXT,
    buyer_message TEXT,
    buyer_x_status_id TEXT,
    escrow_account TEXT
);

-- Indexes
CREATE INDEX idx_offers_agent_id ON offers(agent_id);
CREATE INDEX idx_offers_payment_link_status ON offers(payment_link_status);
CREATE INDEX idx_offers_created_at ON offers(created_at);
```

### Agents Table
```sql
CREATE TABLE agents (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    username TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    is_registered BOOLEAN DEFAULT false
);

-- Indexes
CREATE INDEX idx_agents_username ON agents(username);
CREATE INDEX idx_agents_is_registered ON agents(is_registered);
```

## Relationships

1. **Offers to Agents**
   - One-to-Many relationship
   - An agent can have multiple offers
   - An offer belongs to one agent

## Database Functions

### Get Agent Offers
```sql
CREATE OR REPLACE FUNCTION get_agent_offers(
    p_agent_id TEXT,
    p_status TEXT DEFAULT NULL,
    p_limit INTEGER DEFAULT 10,
    p_offset INTEGER DEFAULT 0
) RETURNS TABLE (
    id UUID,
    payment_amount_sol DECIMAL,
    payment_amount_payai DECIMAL,
    request_description TEXT,
    created_at TIMESTAMP WITH TIME ZONE,
    payment_link_status TEXT
) AS $$
BEGIN
    RETURN QUERY
    SELECT o.id, o.payment_amount_sol, o.payment_amount_payai, o.request_description,
           o.created_at, o.payment_link_status
    FROM offers o
    WHERE o.agent_id = p_agent_id
    AND (p_status IS NULL OR o.payment_link_status = p_status)
    ORDER BY o.created_at DESC
    LIMIT p_limit
    OFFSET p_offset;
END;
$$ LANGUAGE plpgsql;
```

## Security

### Row Level Security (RLS)
```sql
-- Enable RLS on all tables
ALTER TABLE offers ENABLE ROW LEVEL SECURITY;
ALTER TABLE agents ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Public offers are viewable by everyone" ON offers
    FOR SELECT USING (true);

CREATE POLICY "Agents can view their own offers" ON offers
    FOR SELECT USING (agent_id = current_user);

CREATE POLICY "Only authenticated users can create offers" ON offers
    FOR INSERT WITH CHECK (auth.role() = 'authenticated');
```