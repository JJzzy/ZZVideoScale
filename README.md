# ZZVideoScale
这是一款基于OpenGl ES针对视频进行缩放的核心算法实现


项目中的拖拽、缩放功能包含了单指、多指操作，且能让缩放和拖拽顺滑、连续、准确。

通过本文件中的拖拽、缩放手势及其核心算法，会计算出对应的在OpenGl中的中心点坐标，经过OpenGl的处理后就可以实现画面的缩放、拖拽


OpenGl中如何处理当前的缩放倍数、中心点坐标，核心代码如下：


- (void)setPositionWithCoord:(GLfloat *)coord {

    int length = 8; //(int)sizeof(coordVertices)/GLfloat;
    
    if(coord != NULL) {
    
        for(int i = 0; i < length; i++) {
        
            GLfloat value = *(coord);
            
            if(i % 2 == 1) {
            
                //因为纹理坐标系与视频UIKit坐标系上下相反的对应关系，需作处理
                
                value = 1 - value;
                
            }
            
            coordVertices[i] = value;
            
            coord++;
            
        }
        
    }
    
    //    更新放大区域
    
    glVertexAttribPointer(ATTRIB_TEXTURE, 2, GL_FLOAT, 0, 0, coordVertices);
    
    glEnableVertexAttribArray(ATTRIB_TEXTURE);
    
}



/**
 设置放大比例，x,y 是希望在屏幕上放大的区域的中心点
 **/
 
- (void)scaleWithScaleRatio:(GLfloat)scaleRatio x:(GLfloat)x y:(GLfloat)y //设置放大比例

{

    UVILog(@"scaleRation:%f ,x=%f,y=%f", scaleRatio, x, y);
    centerPoint.x = x;
    centerPoint.y = y;
    GLfloat coord[8];
    if(scaleRatio <= 1.0f) {
        centerPoint.x = 0.5f;
        centerPoint.y = 0.5f;
        
        UVDLog(@"scaleRatio==1.0f");
        memcpy(coordVertices, coordVertices_init, sizeof(coord));
        //    更新放大区域
        glVertexAttribPointer(ATTRIB_TEXTURE, 2, GL_FLOAT, 0, 0, coordVertices);
        
        glEnableVertexAttribArray(ATTRIB_TEXTURE);
        mScaleRatio = 1.0f;
        return; //注意，因为此时给了初始值，不需要再调用SetPosition了，直接返回
        
    } else {
        mScaleRatio = scaleRatio;
        //计算比例关系
        float deltaX = (1.0f / scaleRatio) / 2;
        float deltaY = (1.0f / scaleRatio) / 2;
        
        //计算四个点的坐标,给定的点是中心点（x, y），
        float leftX, leftY;
        leftX = x - deltaX; //左上角X
        leftY = y + deltaY; //左上角Y
        
        if(leftX < 0.0f) {
            //X,Y 太靠左,向右移
            leftX = 0.0f;
        }
        else if((x + deltaX  )> 1.0f) {
            //太靠右，向左移。
            leftX = 1.0f - 2*deltaX;
        }
        
        if(leftY > 1.0f) {
            //Y 太靠上
            leftY = 1.0f;
        }
        else if((y - deltaY ) < 0.0f) {
            //太靠下
            leftY = 0.0f + 2*deltaY;
        }
        
        coord[0] = leftX; //左上角
        coord[1] = leftY;
        
        coord[2] = leftX + 2*deltaX; //右上角
        coord[3] = leftY;
        
        coord[4] = leftX; //左下角
        coord[5] = leftY - 2*deltaY;
        
        coord[6] = leftX + 2*deltaX; //右下角
        coord[7] = leftY - 2*deltaY;
        
        for(int i=0;i<8;i++)
        {
            if(coord[i]<0.0f)
            {
                coord[i]=0.0f;
            }
            else if(coord[i]>1.0f)
            {
                coord[i]=1.0f;
            }
        }
        UVILog(@"scaleRatio:%f ,leftX=%f,leftY=%f,deltaX:%f，deltaY:%f", scaleRatio,leftX,leftY,deltaX,deltaY);
        
    }
    
    [self setPositionWithCoord:coord];
}
