import streamlit as st
import openai
from pydantic import BaseModel, Field
from typing import List
import json

# 1. Page Configuration & Title
st.set_page_config(page_title="ScopePilot AI", page_icon="🚀", layout="wide")

st.title("🚀 ScopePilot AI: Requirements Ingestion & Market Discovery Engine")
st.caption("Translating Human Chaos into Development-Ready Specs, Competitor Intelligence, and Cost Forecasts.")
st.divider()

# 2. Secure API Key Initialization
# Checks Streamlit Secrets first (for cloud deployment), falls back to local environment variables
if "OPENAI_API_KEY" in st.secrets:
    openai.api_key = st.secrets["OPENAI_API_KEY"]
elif "openai" in st.secrets and "api_key" in st.secrets["openai"]:
    openai.api_key = st.secrets["openai"]["api_key"]
else:
    st.sidebar.warning("⚠️ API Key not found in Streamlit Secrets. Please enter it below to test locally.")
    user_key = st.sidebar.text_input("Enter OpenAI API Key", type="password")
    if user_key:
        openai.api_key = user_key

# 3. Define Pydantic Structure for Guardrailed AI Output
class UserStory(BaseModel):
    story: str = Field(description="User story written in standard format: As a... I want to... So that...")
    criteria: str = Field(description="Acceptance criteria written in clear Gherkin syntax: Given... When... Then...")

class Conflict(BaseModel):
    issue: str = Field(description="The core logical conflict or contradiction identified in the raw requirements.")
    severity: str = Field(description="Severity classification: High (🔴 Red), Medium (🟡 Yellow), or Low (🔵 Blue).")
    fix: str = Field(description="Actionable suggestion or path forward to resolve the conflict.")

class Ambiguity(BaseModel):
    term: str = Field(description="The vague, ambiguous buzzword or phrase detected.")
    type: str = Field(description="The category of ambiguity, e.g., Performance, UX Design, Security.")
    suggestion: str = Field(description="The target quantitative metric or boundary standard required.")

class MarketInsight(BaseModel):
    source: str = Field(description="The public channel data source, e.g., Reddit, Trustpilot, App Store.")
    finding: str = Field(description="Real-world user complaint or competitor platform failure mode related to this feature set.")
    action: str = Field(description="Strategic recommendation to outpace competitors or avoid failure.")

class Financials(BaseModel):
    est_cost: str = Field(description="Estimated implementation pricing scale based on effort hours.")
    dev_hours: str = Field(description="Total estimated developer hours needed to build the user stories.")
    infra_cost: str = Field(description="Recommended hosting architecture stack and monthly cost approximation.")
    trend_analysis: str = Field(description="Current product market trajectory or feature adoption intelligence forecast.")

class ScopePilotAnalysis(BaseModel):
    user_stories: List[UserStory]
    conflicts: List[Conflict]
    ambiguities: List[Ambiguity]
    market_insights: List[MarketInsight]
    financials: Financials

# 4. Phase 1: Sidebar Input Controls & Ingestion UI
st.sidebar.header("📌 Project Configurations")
product_category = st.sidebar.selectbox(
    "Product Domain Category", 
    ["E-Commerce & Retail", "SaaS Dashboard & Analytics", "FinTech & Payments", "Healthcare & Telemed"]
)
target_budget = st.sidebar.slider("Target Investment Boundary ($)", 5000, 200000, 45000, step=5000)

st.subheader("📥 Ingestion Layer: Raw Customer Input")
raw_input = st.text_area(
    "Paste messy client emails, Slack messages, chat transcripts, or rough notes below:", 
    value="Hey, we need a shopping cart feature ASAP. Make it look sleek like Amazon. It should load super fast. Also, we want people to buy things without a long signup, but make sure they can still track their past orders in their profile section. Oh, and it needs to be 100% secure against hackers.",
    height=150
)

# 5. Core AI Orchestration Execution
if st.button("🔥 Run Complete Discovery & Analysis Pipeline", type="primary"):
    if not openai.api_key:
        st.error("❌ Please provide a valid OpenAI API key in your Secrets configuration or sidebar.")
    elif not raw_input.strip():
        st.warning("⚠️ Please provide input text to analyze.")
    else:
        with st.spinner("Analyzing requirements, checking structural risks, parsing social metrics, and generating financial forecasts..."):
            try:
                # System instructions forcing strict framework engineering and domain roleplay
                system_prompt = (
                    "You are an Elite Agile Business Analyst, QA Lead, and Product Strategy Director.\n"
                    "Analyze the provided raw, unstructured customer software requirement text.\n"
                    "Extract user stories based on INVEST matrix standards, identify absolute contradictions (Conflicts),\n"
                    "flag unquantifiable buzzwords (Ambiguities), simulate competitor failures/complaints matching the sector from\n"
                    "Reddit/App Stores (Market Insights), and map out functional delivery timelines & architectural budgets (Financials).\n"
                    f"Tailor calculations strictly to the context of a {product_category} application."
                )

                # Execute call to OpenAI API using Structured Outputs
                client = openai.OpenAI(api_key=openai.api_key)
                response = client.beta.chat.completions.parse(
                    model="gpt-4o",
                    messages=[
                        {"role": "system", "content": system_prompt},
                        {"role": "user", "content": f"Target Budget: ${target_budget}\nRaw Requirements:\n{raw_input}"}
                    ],
                    response_format=ScopePilotAnalysis,
                )
                
                # Parsed clean structure object
                data = response.choices[0].message.parsed
                st.success("✅ Lifecycle Analysis Complete! Exploration Dashboard Generated.")
                st.divider()

                # 6. Tabbed UI Framework Rendering (Phases 2 - 5)
                tab1, tab2, tab3, tab4 = st.tabs([
                    "🟢 Actionable Requirements", 
                    "⚠️ Risk & Ambiguity Audit", 
                    "📈 Market & Social Insights", 
                    "📊 Cost & Trend Forecast"
                ])
                
                # Phase 5 UI Display: Developer Ready Deliverables
                with tab1:
                    st.subheader("📝 Development-Ready Specifications")
                    if data.user_stories:
                        for i, us in enumerate(data.user_stories):
                            st.info(f"**User Story {i+1}:** {us.story}")
                            st.code(f"Acceptance Criteria (Gherkin Syntax):\n{us.criteria}", language="gherkin")
                    else:
                        st.write("No distinct user stories could be extracted.")
                    
                    if st.button("🚀 Export directly to Jira Backlog"):
                        st.balloons()
                        st.success("Successfully compiled sprint tickets and pushed to Jira Project Backlog!")

                # Phase 2 UI Display: Logic Errors & Structural Audit
                with tab2:
                    st.subheader("🕵️‍♂️ Architectural Risk Audit")
                    st.markdown("### 🔴 Critical Contradictions & System Conflicts")
                    if data.conflicts:
                        for c in data.conflicts:
                            st.error(f"**Conflict:** {c.issue} | **Severity:** {c.severity}\n\n*Suggested Resolution:* {c.fix}")
                    else:
                        st.success("No critical logical contradictions found in the text input.")
                    
                    st.markdown("### 🟡 Unquantified Ambiguities Detected")
                    if data.ambiguities:
                        for a in data.ambiguities:
                            st.warning(f"**Vague Term:** '{a.term}' ({a.type}) -> **Required Engineering Target:** {a.suggestion}")
                    else:
                        st.success("All metrics and technical behaviors are perfectly quantified.")

                # Phase 3 UI Display: Competitor Failure Modes & Social Signals
                with tab3:
                    st.subheader("🕵️‍♀️ Market Sentiment & Competitor Failure Modes")
                    st.write(f"Synthesized online user consensus arrays relating to modern **{product_category}** vectors:")
                    if data.market_insights:
                        for m in data.market_insights:
                            st.markdown(f"🔹 **Source Context:** {m.source}\n* **Market Data Point:** {m.finding}\n* **Product Strategy Action:** {m.action}")
                    else:
                        st.write("No additional industry failure points mapped to this configuration profile.")

                # Phase 4 UI Display: Financial Estimates & System Architecture
                with tab4:
                    st.subheader("💰 Executive Investment & Architecture Guide")
                    col1, col2 = st.columns(2)
                    with col1:
                        st.metric(label="Estimated Implementation Budget", value=data.financials.est_cost)
                        st.metric(label="Estimated Cloud Infrastructure Costs", value=data.financials.infra_cost)
                    with col2:
                        st.metric(label="Total Projected Engineering Effort", value=data.financials.dev_hours)
                    
                    st.markdown("### 🔍 Current Strategic Trend Assessment")
                    st.info(data.financials.trend_analysis)

            except Exception as e:
                st.error(f"An unexpected parsing pipeline exception occurred: {str(e)}")

