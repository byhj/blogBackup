title: Direct3D12-HelloWorld
date: 2015-12-18 12:37:51
tags:
 - Direct3D
 - D3D12
---

# Introduction
Direct 3D是基于微软的通用对象模式COM（Common Object Mode）的3D图形API。它是由微软（Microsoft）一手树立的3D API规范，微软公司拥有该库版权，它所有的语法定义包含在微软提供的程序开发组件的帮助文件、源代码中。Direct3D 12是新一代的图形API，使得图形开发人员能得到更加多的功能和效率。

<!--more-->

# Create A Framework
　　建立一个框架类

```
#ifndef D3DAPP_H
#define D3DAPP_H

#define WIN32_LEAN_AND_MEAN

#include <windows.h>
#include <windowsx.h>
#include <d2d1_3.h>
#include <dwrite.h>
#include <d3d11on12.h>
#include <d3d12.h>
#include <dxgi1_4.h>
#include <D3Dcompiler.h>
#include <DirectXMath.h>
#include "d3dx12.h"

#include <string>
#include <wrl.h>

namespace byhj  
{

namespace d3d
{


class App
{
public:
	App() :m_AppName(L"DirectX12: "), m_WndClassName(L"D3DWindow")
	{

	}
	virtual ~App() {}

	void InitApp();
	int Run();
	LRESULT CALLBACK MessageHandler(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam);

	virtual	void v_Init()     = 0;
	virtual void v_Update()   = 0;
	virtual void v_Render()   = 0;
	virtual void v_Shutdown() = 0;

	// Convenience overrides for handling mouse input.
	virtual void v_OnMouseDown(WPARAM btnState, int x, int y)  { }
	virtual void v_OnMouseUp(WPARAM btnState, int x, int y)    { }
	virtual void v_OnMouseMove(WPARAM btnState, int x, int y)  { }
	virtual void v_OnMouseWheel(WPARAM btnState, int x, int y) { }

protected:

	int   m_ScreenWidth;
	int   m_ScreenHeight;
	float m_ScreenFar;
	float m_ScreenNear;
	int   m_PosX;
	int   m_PosY;

	LPCTSTR m_AppName;
	LPCTSTR m_WndClassName;

	//void      GetVideoCardInfo(char &, int &);
	HINSTANCE GetAppInst() const { return m_hInstance; }
	HWND      GetHwnd()    const { return m_hWnd; }
	float     GetAspect()  const { return static_cast<float>(m_ScreenWidth)
		                                  / static_cast<float>(m_ScreenHeight); }

private:
	bool init_window();

private:
	HINSTANCE m_hInstance;
	HWND      m_hWnd;
};

}

}
#endif
```

框架类的实现

```
#include "App.h"

namespace byhj
{

namespace d3d
{

static LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);
static App *AppHandle = 0;


//主程序循环,首先创建一个windows窗口类，然后进入渲染循环，每次循环先处理消息，然后更新渲染资源然后显示新的图像

int App::Run()
{
	InitApp();

	MSG msg;
	ZeroMemory(&msg, sizeof(MSG));
	while (true)
	{
		if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE) )
		{
			if(msg.message == WM_QUIT)
				break;
			TranslateMessage(&msg);
			DispatchMessage(&msg);
		}
		else
		{
			v_Update();
			v_Render();
		}

	}

	v_Shutdown();
	return (int)msg.wParam;
}

void App::InitApp()
{
	init_window();

	v_Init();

}


bool App::init_window()
{
	//Set the window in the middle of screen
	m_ScreenWidth = GetSystemMetrics(SM_CXSCREEN) * 0.75;
	m_ScreenHeight = GetSystemMetrics(SM_CYSCREEN) * 0.75;
	m_PosX = (GetSystemMetrics(SM_CXSCREEN) - m_ScreenWidth)  / 2;
	m_PosY = (GetSystemMetrics(SM_CYSCREEN) - m_ScreenHeight) / 2;
	m_ScreenNear = 0.1f;
	m_ScreenFar  = 1000.0f;

	AppHandle = this;
	m_hInstance = GetModuleHandle(NULL);

	WNDCLASSEX wc;
	wc.cbSize = sizeof(WNDCLASSEX);
	wc.style = CS_HREDRAW | CS_VREDRAW | CS_OWNDC;
	wc.lpfnWndProc = WndProc;
	wc.cbClsExtra = NULL;
	wc.cbWndExtra = NULL;
	wc.hInstance = m_hInstance;
	wc.hIcon = LoadIcon(NULL, IDI_WINLOGO);
	wc.hCursor = LoadCursor(NULL, IDC_ARROW);
	wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 2);
	wc.lpszMenuName = NULL;
	wc.lpszClassName = m_WndClassName;
	wc.hIconSm = LoadIcon(NULL, IDI_WINLOGO);

	if (!RegisterClassEx(&wc))
	{
		MessageBox(NULL, L"Registering Class Failded",	L"Error", MB_OK | MB_ICONERROR);
		return 1;
	}

	//Create the window and show

	m_hWnd = CreateWindowEx(
		NULL,	           
		m_WndClassName,
		m_AppName,
		WS_OVERLAPPEDWINDOW,
		m_PosX, m_PosY,
		m_ScreenWidth,
		m_ScreenHeight,
		NULL,
		NULL,
		m_hInstance,
		NULL
		);

	if (!m_hWnd )
	{
		MessageBox(NULL, L"Creating Window Failed", L"Error", MB_OK | MB_ICONERROR);
		return 1;
	}

	ShowWindow(m_hWnd, SW_SHOW);

	return true;
}

//处理消息的回调函数

LRESULT CALLBACK App::MessageHandler(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
	switch (uMsg)
	{
	case WM_KEYDOWN:
		{
			if(wParam == VK_ESCAPE)
				PostMessage(GetHwnd(), WM_DESTROY, 0, 0);
		}
	case WM_LBUTTONDOWN:
	case WM_MBUTTONDOWN:
	case WM_RBUTTONDOWN:
		v_OnMouseDown(wParam, GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam));
		return 0;
	case WM_LBUTTONUP:
	case WM_MBUTTONUP:
	case WM_RBUTTONUP:
		v_OnMouseUp(wParam, GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam));
		return 0;
	case WM_MOUSEMOVE:
		v_OnMouseMove(wParam, GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam));
		return 0;
	case WM_MOUSEWHEEL:
		v_OnMouseWheel(wParam, GET_WHEEL_DELTA_WPARAM(wParam), GET_Y_LPARAM(lParam));
	default:
		{
			return DefWindowProc(hWnd, uMsg, wParam, lParam);
		}
	}

}

LRESULT CALLBACK WndProc(HWND hwnd, UINT umessage, WPARAM wparam, LPARAM lparam)
{
	switch(umessage)
	{
	case WM_DESTROY:
		{
			PostQuitMessage(0);
			return 0;
		}
	case WM_CLOSE:
		{
			PostQuitMessage(0);		
			return 0;
		}
		// All other messages pass to the message handler in the system class.
	default:
		{
			return AppHandle->MessageHandler(hwnd, umessage, wparam, lparam);
		}
	}
}

}

}

```

# Create a Triangle


```
#ifndef RENDERSYSTEM_H
#define RENDERSYSTEM_H

#include "d3d/App.h"
#include "triangle.h"

#include <d3d12.h>
#include <wrl.h>
#include <DirectXMath.h>
#include <dxgi1_4.h>
#include <windows.h>

using namespace DirectX;
using namespace Microsoft::WRL;

namespace byhj
{

class RenderSysem : public byhj::d3d::App
{
public:

	void v_Init();
	void v_Update();
	void v_Render();
	void v_Shutdown();

private:

	void PopulateCommandList();
	void WaitForPreviousFrame();
	void LoadAssets();
	void LoadPipeline();

	static const UINT FrameCount = 2;
	D3D12_VIEWPORT m_Viewport;
	D3D12_RECT m_ScissorRect;

	ComPtr<IDXGISwapChain3>           m_pSwapChain;
	ComPtr<ID3D12Device>              m_pD3D12Device;
	ComPtr<ID3D12Fence>               m_pD3D12Fence;
	ComPtr<ID3D12Resource>            m_pRenderTargets[FrameCount];
	ComPtr<ID3D12CommandAllocator>    m_pCommandAllocator;
	ComPtr<ID3D12CommandQueue>        m_pCommandQueue;
	ComPtr<ID3D12DescriptorHeap>      m_pRTVHeap;
	ComPtr<ID3D12GraphicsCommandList> m_pCommandList;

	UINT   m_RTVDescriptorSize;
	UINT   m_FrameIndex;
	HANDLE m_FenceEvent;
	UINT64 m_FenceValue;

	Triangle m_Triangle;
};


}

#endif
```

RenderSystem实现

```
#include "RenderSystem.h"

#include "d3d/Utility.h"
#include "d3dx12.h"
#include <d3dcompiler.h>

namespace byhj
{

void RenderSysem::v_Init()
{
	m_Viewport.Width = static_cast<float>(m_ScreenWidth);
	m_Viewport.Height = static_cast<float>(m_ScreenHeight);
	m_Viewport.MaxDepth = 1.0f;

	m_ScissorRect.right = static_cast<LONG>(m_ScreenWidth);
	m_ScissorRect.bottom = static_cast<LONG>(m_ScreenHeight);

	LoadPipeline();
	LoadAssets();
	m_Triangle.init_buffer(m_pD3D12Device);
	m_Triangle.init_shader(m_pD3D12Device);
}

void RenderSysem::v_Update()
{

}

void RenderSysem::v_Render()
{
	// Record all the commands we need to render the scene into the command list.
	PopulateCommandList();

	// Execute the command list.
	ID3D12CommandList *ppCommandLists[] ={ m_pCommandList.Get() };
	m_pCommandQueue->ExecuteCommandLists(_countof(ppCommandLists), ppCommandLists);

	// Present the frame.
	ThrowIfFailed(m_pSwapChain->Present(1, 0));

	WaitForPreviousFrame();
}

void RenderSysem::v_Shutdown()
{


	WaitForPreviousFrame();

	//ClouseHandle(m_FenceEvent);
}

void RenderSysem::LoadPipeline()
{

#ifdef _DEBUG
	//Enable the D3D12 debug layer
	ComPtr<ID3D12Debug> pDebugController;
	if (SUCCEEDED(D3D12GetDebugInterface(IID_PPV_ARGS(&pDebugController))))
	{
		pDebugController->EnableDebugLayer();
	}
#endif

	ComPtr<IDXGIFactory4> pFactory;
	ThrowIfFailed(CreateDXGIFactory1(IID_PPV_ARGS(&pFactory)));

	bool UseWarpDevice = false;

	if (UseWarpDevice)
	{
		ComPtr<IDXGIAdapter> pWarpAdapter;
		ThrowIfFailed(pFactory->EnumWarpAdapter(IID_PPV_ARGS(&pWarpAdapter)));
		ThrowIfFailed(D3D12CreateDevice(pWarpAdapter.Get(), D3D_FEATURE_LEVEL_11_0, IID_PPV_ARGS(&m_pD3D12Device)));
	}
	else
	{
		ThrowIfFailed(D3D12CreateDevice(nullptr, D3D_FEATURE_LEVEL_11_0, IID_PPV_ARGS(&m_pD3D12Device)));
	}

///////////////////////	//Describe and Create the command queue//////////////////////////////////////////////

	D3D12_COMMAND_QUEUE_DESC queueDesc ={};
	queueDesc.Flags = D3D12_COMMAND_QUEUE_FLAG_NONE;
	queueDesc.Type  = D3D12_COMMAND_LIST_TYPE_DIRECT;
	ThrowIfFailed(m_pD3D12Device->CreateCommandQueue(&queueDesc, IID_PPV_ARGS(&m_pCommandQueue)));

	//Describe and Create the swap chain
	DXGI_SWAP_CHAIN_DESC swapChainDesc ={};
	swapChainDesc.BufferCount       = FrameCount;
	swapChainDesc.BufferDesc.Width  = m_ScreenWidth;
	swapChainDesc.BufferDesc.Height = m_ScreenHeight;
	swapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
	swapChainDesc.BufferUsage       = DXGI_USAGE_RENDER_TARGET_OUTPUT;
	swapChainDesc.SwapEffect        = DXGI_SWAP_EFFECT_FLIP_DISCARD;
	swapChainDesc.OutputWindow      = GetHwnd();
	swapChainDesc.SampleDesc.Count  = 1;
	swapChainDesc.Windowed          = TRUE;

	ComPtr<IDXGISwapChain> pSwapChain;
	ThrowIfFailed(pFactory->CreateSwapChain(
		m_pCommandQueue.Get(),		// Swap chain needs the queue so that it can force a flush on it.
		&swapChainDesc,
		&pSwapChain
		)
		);

	ThrowIfFailed(pSwapChain.As(&m_pSwapChain));
	m_FrameIndex = m_pSwapChain->GetCurrentBackBufferIndex();

///////////////////////// Create Decriptor heaps /////////////////////////////////////////////


	{
		D3D12_DESCRIPTOR_HEAP_DESC rtvHeapDesc ={};
		rtvHeapDesc.NumDescriptors = FrameCount;
		rtvHeapDesc.Type           = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;
		rtvHeapDesc.Flags          = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;

		ThrowIfFailed(m_pD3D12Device->CreateDescriptorHeap(&rtvHeapDesc, IID_PPV_ARGS(&m_pRTVHeap)));

		m_RTVDescriptorSize = m_pD3D12Device->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);

	}

	//Create frame resources
	{
		CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_pRTVHeap->GetCPUDescriptorHandleForHeapStart());

		// Create a RTV for each frame.
		for (UINT n = 0; n < FrameCount; n++)
		{
			ThrowIfFailed(m_pSwapChain->GetBuffer(n, IID_PPV_ARGS(&m_pRenderTargets[n])));
			m_pD3D12Device->CreateRenderTargetView(m_pRenderTargets[n].Get(), nullptr, rtvHandle);
			rtvHandle.Offset(1, m_RTVDescriptorSize);
		}
	}
	//The command allocator is going to be used for allocating memory for the list of commands that we send to the GPU each frame to render graphics.
	ThrowIfFailed(m_pD3D12Device->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&m_pCommandAllocator)));

}


void RenderSysem::LoadAssets()
{
	// Create the command list.
	ThrowIfFailed(m_pD3D12Device->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, m_pCommandAllocator.Get(), nullptr, IID_PPV_ARGS(&m_pCommandList)));

	// Command lists are created in the recording state, but there is nothing
	// to record yet. The main loop expects it to be closed, so close it now.
	ThrowIfFailed(m_pCommandList->Close());

	// Create synchronization objects.
	{
		// We use the fence as a signaling mechanism to notify us when the GPU is completely done rendering the command list that we submitted via the command queue
		ThrowIfFailed(m_pD3D12Device->CreateFence(0, D3D12_FENCE_FLAG_NONE, IID_PPV_ARGS(&m_pD3D12Fence)));
		m_FenceValue = 1;

		// Create an event handle to use for frame synchronization.
		m_FenceEvent = CreateEventEx(nullptr, FALSE, FALSE, EVENT_ALL_ACCESS);
		if (m_FenceEvent == nullptr)
		{
			ThrowIfFailed(HRESULT_FROM_WIN32(GetLastError()));
		}

		// Wait for the command list to execute; we are reusing the same command
		// list in our main loop but for now, we just want to wait for setup to
		// complete before continuing.
		WaitForPreviousFrame();
	}
}


void RenderSysem::PopulateCommandList()
{
	// Command list allocators can only be reset when the associated
	// command lists have finished execution on the GPU; apps should use
	// fences to determine GPU execution progress.
	ThrowIfFailed(m_pCommandAllocator->Reset());

	// However, when ExecuteCommandList() is called on a particular command
	// list, that command list can then be reset at any time and must be before
	// re-recording.
	auto pPipelineState = m_Triangle.GetPipelineState();
	ThrowIfFailed(m_pCommandList->Reset(m_pCommandAllocator.Get(), pPipelineState.Get()));
	auto pRootSignature = m_Triangle.GetRootSignature();
	m_pCommandList->SetGraphicsRootSignature(pRootSignature.Get());

	m_pCommandList->RSSetViewports(1, &m_Viewport);
	m_pCommandList->RSSetScissorRects(1, &m_ScissorRect);
	m_pCommandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_pRenderTargets[m_FrameIndex].Get(), D3D12_RESOURCE_STATE_PRESENT, D3D12_RESOURCE_STATE_RENDER_TARGET));

	CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(m_pRTVHeap->GetCPUDescriptorHandleForHeapStart(), m_FrameIndex, m_RTVDescriptorSize);
	m_pCommandList->OMSetRenderTargets(1, &rtvHandle, FALSE, nullptr);

	// Record commands.
	const float clearColor[] ={ 0.0f, 0.2f, 0.4f, 1.0f };
	m_pCommandList->ClearRenderTargetView(rtvHandle, clearColor, 0, nullptr);

	m_Triangle.Render(m_pCommandList);

	//pCIndicate that the back buffer will now be used to present.
	m_pCommandList->ResourceBarrier(1, &CD3DX12_RESOURCE_BARRIER::Transition(m_pRenderTargets[m_FrameIndex].Get(), D3D12_RESOURCE_STATE_RENDER_TARGET, D3D12_RESOURCE_STATE_PRESENT));

	ThrowIfFailed(m_pCommandList->Close());


}


void  RenderSysem::WaitForPreviousFrame()
{
	// WAITING FOR THE FRAME TO COMPLETE BEFORE CONTINUING IS NOT BEST PRACTICE.
	// This is code implemented as such for simplicity. More advanced samples
	// illustrate how to use fences for efficient resource usage.

	// Signal and increment the fence value.
	const UINT64 fence = m_FenceValue;
	ThrowIfFailed(m_pCommandQueue->Signal(m_pD3D12Fence.Get(), fence));
	m_FenceValue++;

	// Wait until the previous frame is finished.
	if (m_pD3D12Fence->GetCompletedValue() < fence)
	{
		ThrowIfFailed(m_pD3D12Fence->SetEventOnCompletion(fence, m_FenceEvent));
		WaitForSingleObject(m_FenceEvent, INFINITE);
	}

	m_FrameIndex = m_pSwapChain->GetCurrentBackBufferIndex();
}

}

```

Triangle部分

```
#ifndef Triangle_H
#define Triangle_H

#include <d3d12.h>
#include <DirectXMath.h>
#include <wrl.h>

using namespace DirectX;
using namespace Microsoft::WRL;

namespace byhj
{

class Triangle
{
public:

	void Init();
	void Update();
	void Render(ComPtr<ID3D12GraphicsCommandList> pCommandList);
	void Shutdown();

	ComPtr<ID3D12PipelineState> GetPipelineState()
	{
		return m_pPipelineState;
	}
	ComPtr<ID3D12RootSignature> GetRootSignature()
	{
		return m_pRootSignature;
	}
	void init_buffer(ComPtr<ID3D12Device> pD3D12Device);
	void init_shader(ComPtr<ID3D12Device> pD3D12Device);
private:


	struct Vertex
	{
		XMFLOAT3 position;
		XMFLOAT4 color;
	};
	ComPtr<ID3D12PipelineState>  m_pPipelineState;
	ComPtr<ID3D12RootSignature>  m_pRootSignature;
	ComPtr<ID3D12Resource>       m_pVertexBuffer;
	D3D12_VERTEX_BUFFER_VIEW     m_VertexBufferView;
};

}
#endif

```

Triangle实现部分

```
#include "Triangle.h"
#include "d3dx12.h"
#include "d3d/Utility.h"
#include <d3dcompiler.h>

namespace byhj
{

void Triangle::Init()
{

}

void Triangle::Update()
{

}

void Triangle::Render(ComPtr<ID3D12GraphicsCommandList> pCommandList)
{
	pCommandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);
	pCommandList->IASetVertexBuffers(0, 1, &m_VertexBufferView);
	pCommandList->DrawInstanced(3, 1, 0, 0);
}

void Triangle::Shutdown()
{


}

void Triangle::init_buffer(ComPtr<ID3D12Device> pD3D12Device)
{

	// Create the vertex buffer.
	{
		// Define the geometry for a triangle.
		Vertex triangleVertices[] =
		{
			{ {  0.0f,  0.5f, 0.0f }, { 1.0f, 0.0f, 0.0f, 1.0f } },
			{ {  0.5f, -0.5f, 0.0f }, { 0.0f, 1.0f, 0.0f, 1.0f } },
			{ { -0.5f, -0.5f, 0.0f }, { 0.0f, 0.0f, 1.0f, 1.0f } }
		};

		const UINT vertexBufferSize = sizeof(triangleVertices);

		// Note: using upload heaps to transfer static data like vert buffers is not
		// recommended. Every time the GPU needs it, the upload heap will be marshalled
		// over. Please read up on Default Heap usage. An upload heap is used here for
		// code simplicity and because there are very few verts to actually transfer.
		ThrowIfFailed(pD3D12Device->CreateCommittedResource(
			&CD3DX12_HEAP_PROPERTIES(D3D12_HEAP_TYPE_UPLOAD),
			D3D12_HEAP_FLAG_NONE,
			&CD3DX12_RESOURCE_DESC::Buffer(vertexBufferSize),
			D3D12_RESOURCE_STATE_GENERIC_READ,
			nullptr,
			IID_PPV_ARGS(&m_pVertexBuffer)));

		// Copy the triangle data to the vertex buffer.
		UINT8* pVertexDataBegin;
		ThrowIfFailed(m_pVertexBuffer->Map(0, nullptr, reinterpret_cast<void**>(&pVertexDataBegin)));
		memcpy(pVertexDataBegin, triangleVertices, sizeof(triangleVertices));
		m_pVertexBuffer->Unmap(0, nullptr);

		// Initialize the vertex buffer view.
		m_VertexBufferView.BufferLocation = m_pVertexBuffer->GetGPUVirtualAddress();
		m_VertexBufferView.StrideInBytes = sizeof(Vertex);
		m_VertexBufferView.SizeInBytes = vertexBufferSize;
	}
}


void Triangle::init_shader(ComPtr<ID3D12Device> pD3D12Device)
{
	//Create empty root signature
	{
		CD3DX12_ROOT_SIGNATURE_DESC rootSignatureDesc;
		rootSignatureDesc.Init(0, nullptr, 0, nullptr, D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT);

		ComPtr<ID3DBlob> signature;
		ComPtr<ID3DBlob> error;
		ThrowIfFailed(D3D12SerializeRootSignature(&rootSignatureDesc, D3D_ROOT_SIGNATURE_VERSION_1, &signature, &error));
		ThrowIfFailed(pD3D12Device->CreateRootSignature(0, signature->GetBufferPointer(), signature->GetBufferSize(), IID_PPV_ARGS(&m_pRootSignature)));

	}

	// Create the pipeline state, which includes compiling and loading shaders.
	{
		ComPtr<ID3DBlob> vertexShader;
		ComPtr<ID3DBlob> pixelShader;

#ifdef _DEBUG
		// Enable better shader debugging with the graphics debugging tools.
		UINT compileFlags = D3DCOMPILE_DEBUG | D3DCOMPILE_SKIP_OPTIMIZATION;
#else
		UINT compileFlags = 0;
#endif

		ThrowIfFailed(D3DCompileFromFile( L"triangle.vsh", nullptr, nullptr, "VS_MAIN", "vs_5_0", compileFlags, 0, &vertexShader, nullptr));
		ThrowIfFailed(D3DCompileFromFile( L"triangle.psh", nullptr, nullptr, "PS_MAIN", "ps_5_0", compileFlags, 0, &pixelShader, nullptr));

		// Define the vertex input layout.
		D3D12_INPUT_ELEMENT_DESC inputElementDescs[] =
		{
			{ "POSITION", 0, DXGI_FORMAT_R32G32B32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 },
			{ "COLOR", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 12, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 }
		};

		// Describe and create the graphics pipeline state object (PSO).
		D3D12_GRAPHICS_PIPELINE_STATE_DESC psoDesc ={};
		psoDesc.InputLayout ={ inputElementDescs, _countof(inputElementDescs) };
		psoDesc.pRootSignature = m_pRootSignature.Get();
		psoDesc.VS ={ reinterpret_cast<UINT8*>(vertexShader->GetBufferPointer()), vertexShader->GetBufferSize() };
		psoDesc.PS ={ reinterpret_cast<UINT8*>(pixelShader->GetBufferPointer()), pixelShader->GetBufferSize() };
		psoDesc.RasterizerState = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);
		psoDesc.BlendState = CD3DX12_BLEND_DESC(D3D12_DEFAULT);
		psoDesc.DepthStencilState.DepthEnable = FALSE;
		psoDesc.DepthStencilState.StencilEnable = FALSE;
		psoDesc.SampleMask = UINT_MAX;
		psoDesc.PrimitiveTopologyType = D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE;
		psoDesc.NumRenderTargets = 1;
		psoDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;
		psoDesc.SampleDesc.Count = 1;
		ThrowIfFailed(pD3D12Device->CreateGraphicsPipelineState(&psoDesc, IID_PPV_ARGS(&m_pPipelineState)));
	}

}

}
```

# Shader
在这里Vertex Shader和Pixel Shader只是简单的传递顶点和颜色数据

## Vertex Shader

```
struct VS_IN
{
  float4 Pos  : POSITION;
  float4 Color: COLOR0;
};

struct VS_OUT
{
  float4 Pos   : SV_POSITION;
  float4 Color : COLOR0;
};

VS_OUT VS_MAIN( VS_IN vs_in)
{
  VS_OUT vs_out;
  vs_out.Pos   = vs_in.Pos;
  vs_out.Color = vs_in.Color;

  return vs_out;
}
```

## Pixel Shader

```
struct VS_OUT
{
  float4 Pos   : SV_POSITION;
  float4 Color : COLOR0;
};


float4 PS_MAIN( VS_OUT vs_out) :SV_TARGET
{
  return vs_out.Color;
}
```
