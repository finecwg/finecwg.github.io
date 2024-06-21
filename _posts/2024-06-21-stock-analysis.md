---
layout: single
title: "Building a Sophisticated Stock Analysis Bot with CrewAI and Telegram"
categories: computer-science
tags: [crewAI, stock analysis]
toc: true
toc_sticky: true
toc_label: "Stock Analysis Bot"
author_profile: false
---

In today's fast-paced financial markets, having access to comprehensive and timely stock analysis is crucial for making informed investment decisions. This blog post explores the implementation of an advanced stock analysis bot that leverages the power of CrewAI, large language models (LLMs), and various financial APIs to provide in-depth insights on demand.

This project was inspired by the concepts presented in this [YouTube video](https://www.youtube.com/watch?v=-59bKxwir5Q), which served as a starting point for our more advanced implementation.

## Architecture Overview

Our stock analysis bot is built on a multi-agent system using the CrewAI framework. It integrates with Telegram for user interaction and employs several key components:

1. **CrewAI Agents**: Specialized AI agents for different aspects of stock analysis
2. **LLM Integration**: Utilizes Anthropic's Claude 3.5 Sonnet model for natural language processing
3. **Financial Data Tools**: Incorporates yfinance for retrieving stock data
4. **Web Scraping**: Uses a custom tool for gathering additional online information
5. **Telegram Bot**: Provides a user-friendly interface for interacting with the system

Let's dive into each component and explore how they work together to create a powerful stock analysis tool.

## CrewAI Agents and Tasks

The core of our system is built around four specialized CrewAI agents:

1. **Researcher**: Gathers and analyzes market sentiment, news, and trends
2. **Technical Analyst**: Conducts in-depth technical analysis of stock price movements
3. **Financial Analyst**: Assesses company financial health and performance
4. **Hedge Fund Manager**: Synthesizes insights and makes investment recommendations

Each agent is assigned specific tasks that align with their expertise:

```python
research = Task(
    description="Conduct a comprehensive analysis of the latest news, market sentiment, and trends...",
    agent=researcher,
    # ...
)

technical_analysis = Task(
    description="Perform an in-depth technical analysis of {company}'s stock price movements...",
    agent=technical_analyst,
    # ...
)

financial_analysis = Task(
    description="Conduct a thorough analysis of {company}'s financial health and performance...",
    agent=financial_analyst,
    # ...
)

investment_recommendation = Task(
    description="Based on the comprehensive research, technical analysis, and financial analysis reports...",
    agent=hedge_fund_manager,
    # ...
)
```

These tasks are designed to produce detailed reports and recommendations, which are then synthesized into a final investment recommendation.

## LLM Integration

The system leverages Anthropic's Claude 3.5 Sonnet model for natural language processing. This advanced LLM powers the decision-making and language generation capabilities of our agents:

```python
llm_claude = ChatAnthropic(
    model="claude-3-5-sonnet-20240620",
    temperature=0.2,
    timeout=None,
    max_retries=2,
    api_key=ANTHROPIC_API_KEY
)
```

By using a state-of-the-art LLM, we ensure that our agents can understand complex financial concepts, generate human-like responses, and provide nuanced analysis.

## Financial Data Tools

To access real-time and historical stock data, we utilize the `yfinance` library. Custom tools are implemented to retrieve various types of financial information:

```python
@tool("Stock Price")
def stock_price(ticker):
    ticker = yf.Ticker(ticker)
    return ticker.history(period="1mo")

@tool("Income statement")
def income_stmt(ticker):
    ticker = yf.Ticker(ticker)
    income_statement = ticker.income_stmt
    # ...

@tool("Balance Sheet")
def balance_sheet(ticker):
    ticker = yf.Ticker(ticker)
    return ticker.balance_sheet

@tool("Insider Transactions")
def insider_transactions(ticker):
    ticker = yf.Ticker(ticker)
    return ticker.insider_transactions
```

These tools allow our agents to access a wide range of financial data, from stock prices to detailed financial statements and insider trading information.

## Web Scraping

To supplement the financial data with additional context, we incorporate a web scraping tool:

```python
scrape_tool = ScrapeWebsiteTool()
```

This tool enables our Researcher agent to gather relevant information from various online sources, providing a more comprehensive view of the market landscape and company-specific news.

## Telegram Bot Integration

The user interface for our stock analysis bot is implemented using the Telegram Bot API. This allows users to easily request stock analysis through a familiar messaging platform:

```python
async def handle_message(update: Update, context: CallbackContext) -> None:
    query = update.message.text
    stock_qa = StockQA()
    response = stock_qa.get_result(query)

    # Split the response into chunks of 4096 characters (Telegram's message limit)
    chunks = [response[i:i+4096] for i in range(0, len(response), 4096)]

    # Send each chunk as a separate message
    for chunk in chunks:
        await update.message.reply_text(chunk)
```

The `handle_message` function processes user queries, initiates the CrewAI analysis pipeline, and returns the results in digestible chunks that conform to Telegram's message size limits.

## Putting It All Together

The `StockQA` class serves as the orchestrator for our multi-agent system:

```python
class StockQA:
    def __init__(self):
        self.crew = Crew(
            tasks=[research, technical_analysis, financial_analysis, investment_recommendation],
            agents=[researcher, technical_analyst, financial_analyst, hedge_fund_manager],
            verbose=2,
        )

    def get_result(self, query):
        result = self.crew.kickoff(
            inputs={
                "company": query
            }
        )
        return result
```

When a user submits a query through Telegram, the `StockQA` instance initializes the CrewAI pipeline, coordinating the efforts of our specialized agents to produce a comprehensive stock analysis.

## Complete Code

```python
# https://www.youtube.com/watch?v=-59bKxwir5Q

import os
from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())

GROQ_API_KEY = os.environ['GROQ_API_KEY']
TELEGRAM_TOKEN = os.environ['TELEGRAM_TOKEN']
ANTHROPIC_API_KEY = os.environ['ANTHROPIC_API_KEY']

from crewai import Agent, Task, Crew
from crewai_tools import tool, ScrapeWebsiteTool

from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, CallbackContext

import yfinance as yf

from groq import Groq
from langchain_groq import ChatGroq
from langchain_anthropic import ChatAnthropic

#*-----------------LLM-----------------*#

# llm = ChatAnthropic(
#     model="claude-3-sonnet-20240229",
#     temperature=0.2,
#     timeout=None,
#     max_retries=2,
#     api_key=ANTHROPIC_API_KEY
# )

llm_claude = ChatAnthropic(
    model="claude-3-5-sonnet-20240620",
    temperature=0.2,
    timeout=None,
    max_retries=2,
    api_key=ANTHROPIC_API_KEY
)

#*-----------------Tools-----------------*#

@tool("Stock News")
def stock_news(ticker):
    """
    Useful to get news about a stock.
    The input should be a ticker, for example AAPL, NET.
    """
    ticker = yf.Ticker(ticker)
    return ticker.news

scrape_tool = ScrapeWebsiteTool()

@tool("Stock Price")
def stock_price(ticker):
    """
    Useful to get stock price data.
    The input should be a ticker, for example AAPL, NET.
    """
    ticker = yf.Ticker(ticker)
    return ticker.history(period = "1mo")

@tool("Income statement")
def income_stmt(ticker):
    """
    Useful to get the income statement of a company.
    The input to this tool should be a ticker, for example AAPL, NET.
    """
    ticker = yf.Ticker(ticker)
    income_statement = ticker.income_stmt

    # Example of checking if the DataFrame is empty
    if income_statement.empty:
        return "No income statement data available."

    return income_statement

@tool("Balance Sheet")
def balance_sheet(ticker):
    """
    Useful to get the balance sheet of a company.
    The input to this tool should be a ticker, for example AAPL, NET.
    """
    ticker = yf.Ticker(ticker)
    return ticker.balance_sheet

@tool("Insider Transactions")
def insider_transactions(ticker):
    """
    Useful to get insider transactions of a stock.
    The input to this tool should be a ticker, for example AAPL, NET.
    """
    ticker = yf.Ticker(ticker)
    return ticker.insider_transactions


#*-----------------Agents-----------------*#

researcher = Agent(
    role = "Researcher",
    goal = """
        Gather and analyze comprehensive data from various reliable sources to
        provide an in-depth overview of the market sentiment, news, and trends
        surrounding a specific stock. Identify key insights and potential risks
        or opportunities that could impact the stock's performance.
    """,
    backstory = """
        You're a highly skilled researcher with a keen eye for detail and a talent
        for identifying crucial information. You have extensive experience in
        gathering and interpreting data from a wide range of reliable sources,
        including financial reports, news articles, and industry publications.
        Your ability to synthesize complex information and extract actionable
        insights is invaluable in making well-informed investment decisions.
    """,
    tools = [
        scrape_tool,
        stock_news
    ],
    llm=llm_claude,
    max_iter = 5,
    allow_delegation = False,
    verbose = True,
)
technical_analyst = Agent(
    role = "Technical Analyst",
    goal = """
        Conduct in-depth technical analysis of a stock's price movements, volume,
        and other relevant metrics to identify trends, patterns, and potential
        entry or exit points. Provide clear and actionable insights on key support
        and resistance levels, as well as potential price targets.
    """,
    backstory = """
        As a seasoned technical analyst, you have a proven track record of accurately
        predicting stock price movements using advanced charting techniques and
        technical indicators. Your deep understanding of market dynamics and ability
        to identify critical patterns and trends has earned you a reputation as a
        go-to expert for valuable insights and trading recommendations.
    """,
    tools = [
        stock_price
    ],
    llm=llm_claude,
    max_iter = 5,
    allow_delegation = False,
    verbose = True,
)
financial_analyst = Agent(
    role = "Financial Analyst",
    goal = """
        Conduct a thorough analysis of a company's financial statements, ratios, and
        other relevant metrics to assess its financial health, profitability, and
        growth potential. Identify key strengths, weaknesses, and risks that could
        impact the company's stock performance, and provide well-reasoned recommendations.
    """,
    backstory = """
        As a highly experienced financial analyst, you have a deep understanding of
        financial statements, accounting principles, and valuation methodologies.
        You excel at analyzing a company's financial health, identifying trends, and
        assessing potential risks and opportunities. Your ability to provide clear,
        well-supported recommendations based on a holistic view of a company's
        financial position, market sentiment, and qualitative factors is highly valued
        by your clients.
    """,
    tools = [
        income_stmt,
        balance_sheet,
        insider_transactions
    ],
    llm=llm_claude,
    max_iter = 5,
    allow_delegation = False,
    verbose = True,
)
hedge_fund_manager = Agent(
    role = "Hedge Fund Manager",
    goal = """
        Synthesize insights from the researcher, technical analyst, and financial analyst
        to make well-informed, data-driven investment decisions for a portfolio of stocks.
        Develop and implement effective strategies to maximize returns while managing risk,
        and clearly communicate your rationale and expectations to clients.
    """,
    backstory = """
        As a seasoned hedge fund manager with a proven track record of delivering strong
        returns, you are adept at leveraging insights from a team of expert analysts to
        make strategic investment decisions. Your ability to synthesize complex information,
        identify unique opportunities, and adapt to changing market conditions has earned
        you a reputation as a top-performing fund manager. Your clients trust your judgment
        and appreciate your clear, transparent communication style.
    """,
    llm=llm_claude,
    max_iter = 5,
    allow_delegation = False,
    verbose = True,
)


#*-----------------Tasks-----------------*#

research = Task(
    description = """
        Conduct a comprehensive analysis of the latest news, market sentiment, and trends
        surrounding {company}'s stock. Identify key factors influencing the stock's performance,
        potential risks, and opportunities. Provide a detailed summary of your findings,
        including any notable shifts in sentiment or market perception.
    """,
    agent = researcher,
    expected_output = """
        Your final answer MUST be a well-structured, detailed report summarizing the latest
        news, market sentiment, and trends related to {company}'s stock. The report should
        include key insights, potential risks, and opportunities, as well as any significant
        changes in market perception. Use clear headings, subheadings, and bullet points to
        organize the information effectively.
    """,
)
technical_analysis = Task(
    description = """
        Perform an in-depth technical analysis of {company}'s stock price movements, volume,
        and other relevant metrics. Identify key support and resistance levels, chart patterns,
        and potential entry or exit points. Provide a detailed report of your findings, including
        price targets, stop-loss levels, and any other relevant technical insights.
    """,
    agent = technical_analyst,
    expected_output = """
        Your final answer MUST be a comprehensive technical analysis report for {company}'s stock.
        The report should include:
        1. Key support and resistance levels
        2. Relevant chart patterns and trend analysis
        3. Potential entry and exit points, along with price targets and stop-loss levels
        4. Any other significant technical insights or observations
        Use charts, graphs, and other visual aids to support your analysis when appropriate.
    """,
)
financial_analysis = Task(
    description = """
        Conduct a thorough analysis of {company}'s financial health and performance using its
        financial statements, ratios, insider trading data, and other relevant metrics. Assess
        the company's profitability, growth potential, and risk factors. Provide a detailed
        report of your findings, highlighting key strengths, weaknesses, and potential impacts
        on the stock's performance.
    """,
    agent = financial_analyst,
    expected_output = """
        Your final answer MUST be a comprehensive financial analysis report for {company}.
        The report should include:
        1. An overview of the company's revenue, earnings, cash flow, and other key financial metrics
        2. Analysis of the company's profitability, liquidity, and solvency ratios
        3. Assessment of the company's growth potential and market position
        4. Identification of key strengths, weaknesses, and risk factors
        5. Discussion of any notable insider trading activity and its potential implications
        Use tables, charts, and other visual aids to present the data effectively.
    """,
)
investment_recommendation = Task(
    description = """
        Based on the comprehensive research, technical analysis, and financial analysis reports
        provided, develop a well-reasoned investment recommendation for {company}'s stock.
        Consider the potential risks and rewards, as well as the overall market conditions and
        the company's competitive position. Provide a clear rationale for your recommendation,
        along with any relevant caveats or considerations.
    """,
    agent = hedge_fund_manager,
    expected_output = """
        Your final answer MUST be a detailed, well-supported investment recommendation for {company}'s
        stock. The recommendation should include:
        1. A clear stance on whether to STRONG BUY, MODERATE BUY, HOLD, MODERATE SELL, or STRONG SELL the stock
        2. A thorough rationale for your recommendation, drawing insights from the research,
        technical analysis, and financial analysis reports
        3. Discussion of potential risks, rewards, and any relevant market or company-specific factors
        4. Any necessary caveats, considerations, or time horizons for your recommendation
        Your recommendation should be well-structured, convincing, and easy to understand for clients.
    """,
    context = [
        research,
        technical_analysis,
        financial_analysis,
    ],
    output_file = "investment_recommendation.md",
)



#*-----------------Crew-----------------*#

class StockQA:
    def __init__(self):
        self.crew = Crew(
            tasks=[
                research,
                technical_analysis,
                financial_analysis,
                investment_recommendation
            ],
            agents=[
                researcher,
                technical_analyst,
                financial_analyst,
                hedge_fund_manager
            ],
            verbose=2,
        )

    def get_result(self, query):
        result = self.crew.kickoff(
            inputs={
                "company": query
            }
        )
        return result


#*-----------------Telegram Bot-----------------*#



async def start(update: Update, context: CallbackContext) -> None:
    await update.message.reply_text('Hi! Ask me about company stock.')

async def handle_message(update: Update, context: CallbackContext) -> None:
    query = update.message.text
    stock_qa = StockQA()
    response = stock_qa.get_result(query)

    # Split the response into chunks of 4096 characters (Telegram's message limit)
    chunks = [response[i:i+4096] for i in range(0, len(response), 4096)]

    # Send each chunk as a separate message
    for chunk in chunks:
        await update.message.reply_text(chunk)

def main() -> None:
    application = Application.builder().token(TELEGRAM_TOKEN).build()

    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    application.run_polling()

if __name__ == '__main__':
    main()

```

<figcaption>Listing: Full implementation of the Stock Analysis Bot, including all imports, agent definitions, tasks, and Telegram integration.</figcaption>

![alt text](/images/2024-06-21-stock-analysis/image.png)
![alt text](/images/2024-06-21-stock-analysis/image-1.png)

<figcaption>Figure: Operation of Telegram Bot</figcaption>

## Conclusion

By combining the power of CrewAI, advanced language models, and various financial data sources, we've created a sophisticated stock analysis bot capable of providing in-depth insights on demand. This system demonstrates the potential of AI-driven financial analysis tools to assist investors in making more informed decisions.

Future enhancements could include:

- Incorporating additional data sources for even more comprehensive analysis
- Implementing sentiment analysis on social media and news articles
- Developing a user feedback system to continually improve the accuracy of recommendations
- Expanding the bot's capabilities to handle portfolio analysis and market-wide trends

As AI and financial technologies continue to evolve, tools like this stock analysis bot will play an increasingly important role in democratizing access to sophisticated financial insights.

## GitHub Repository

[GitHub Repository Link](https://github.com/finecwg/stock_analysis_bot)

For the complete source code, documentation, and setup instructions, please visit my GitHub repository.

_This post was written with the help of Claude 3.5 Sonnet_
